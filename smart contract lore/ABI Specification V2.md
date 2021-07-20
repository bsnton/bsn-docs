SMART CONTRACT LORE

# ABI  规范V2

本文档涵盖了当前ABI版本。它是默认的，但仍然支持ABI v1。

## **ABI v2**

ABI v2引入了一个新的`header`JSON ABI部分，附加参数放在合约函数参数之前。这些参数是合约安全措施的一部分。例如，ABI v1中引入的用于重放攻击保护的`timestamp`现在被定义为`header`部分的一个附加参数。

除了`timestamp`，我们还引入了新的`expire`附加参数。它指定合约不处理消息的时间跨度。

其他一些小的修改

- 公钥成为标头参数之一。
- 签名移动到根单元格。
- Get 方法放置在一个单独的部分，以帮助找到它们，以及其他公共方法。
- 最后一个单元格引用可以被参数序列化使用，如果后面的所有参数都适合当前单元格，则需要引用（`cell`、`bytes`、`map`、`array`类型）。 ABI v1 仅将最后一个单元格引用用于单元格链接。

### **动机**

鉴于附加参数数量的增加，有必要审查它们的定义方式。 `header` 部分应该包括合约期望在所有公共功能的外部入站消息正文中的所有附加参数。 这些参数按照在“header”部分中出现的顺序放置在“function ID”之前的单元格主体中。

公钥成为减少消息大小的可选参数，从而减少转发费用。 每个合约已经有一个公钥，因此不需要在每个消息中包含它。

签名移动到根单元格以减少远期和gas费用。 鉴于从引用读取单元格会消耗 gas，直接从根单元格读取签名更便宜。 除此之外，额外的单元格会增加远期费用。

## **规范**

ABI 为客户端指定消息体布局以进行合约和合约到合约的交互。

## **Message body**

**外部入站消息**

带有编码函数调用的消息正文具有以下格式：

`Maybe(Signature) + Enc(Header) +Function ID+ Enc(Arguments)`

首先是一个可选的签名。 一位标志前缀表示签名存在。 如果它等于‘1’，则在下一个 512 bit中放置一个签名。 否则，省略签名。

然后是编码的标头参数集（所有函数都相同）。

下一个 **_32 bits_** 由函数 ID 组成，用于标识要调用的合约函数。 函数 ID 位于函数签名的 SHA256 散列的前 32 bit内。 对于外部入站消息中的功能 ID，最高位设置为`0`，对于外部出站消息，最高位设置为`1`。


接下来是函数参数。 它们按照当前规范进行编码并存储到根单元或链中的下一个单元。

**注意**：编码参数不能在不同单元格之间拆分。

**外部出站消息**

外部出站消息从函数返回值或发出事件。

返回值被编码并放入消息响应中：

`Function ID+Enc(Return values)`


功能 ID 的最高位设置为 1。

事件编码如下：

`Event ID + Enc(event args)`

`Event ID`  - 事件函数签名的 SHA256 哈希的 32 bit，最高位设置为 0。

**内部消息**

内部消息执行合约间交互； 它们具有以下正文格式：

`Function ID+Enc(Arguments)`

`Function ID` - 32 bit 函数 ID，计算为函数签名的前 32 bit SHA256 哈希值。 函数 ID 的最高位是`0`。 内部消息只包含函数调用而没有响应。

## **消息正文签名**

可以使用加密签名保护消息正文，以识别区块链之外的用户。 在这种情况下，调用该函数的 _外部入站消息_ 携带用户 _私钥_ 签名。 此要求仅适用于 _外部入站消息_ ，因为 _内部入站消息_ 源自区块链，您可以通过 _src address_ 识别调用者。


如果用户不想对消息进行签名，则应将位 0 放置到根单元格的开头并省略签名。

消息正文签名源自签名之后的单元包的表示哈希。

**签名算法**

1. ABI 序列化生成一包包含标题参数、函数 ID 和函数参数的单元格。 根单元中保留 513 个空闲位用于签名和签名标志
2. 使用  _Ed25519_ 算法对包的表示哈希进行签名。
3. Bit `1` 后跟签名的 512 bits，在头参数之前保存到根单元的开头。

## **函数签名 (Function ID)**

以下语法定义了签名：:

- 函数名
- 括号中的输入参数类型列表（输入列表）
- 括号中的返回值类型列表（输出列表）
- ABI版本


单个逗号分隔每个输入参数并返回值类型彼此之间没有空格。

不包括参数和返回值名称。

函数名称、输入和输出列表没有分开，并且紧跟在后面。

如果函数没有输入参数或不返回任何值，则相应的输入或输出列表为空（空括号）。

**函数签名语法**

`function_name(input_type1,input_type2,...,input_typeN)(output_type1,output_type2,...,output_typeM)v2`

**签名计算语法**

`SHA256("function_name(input_type1,input_type2,...,input_typeN)(output_type1,output_type2,...,output_typeM)")v2`

**实现示例**

函数

`func(int64 param1, bool param2) -> uint32`

函数签名

`func(int64,bool)(uint32)v2`

函数哈希

`sha256("func(int64,bool)(uint32)v2") = 0x1354f2c85b50aa84c2f65ebb8cec69aba0aa3269c21e03e142e014e84ea59649`

**function ID** 然后是 `0x1354f2c8` 用于函数调用和 `0x9354f2c8` 用于函数响应

**Event ID**

**Event ID**以与 **function ID** 相同的方式计算，除了事件签名不包含返回值类型列表的情况：`event(int64,bool)v2`

## **编码**

ABI 规范的目标是设计读起来便宜的 ABI 类型，以减少 gas 消耗和 gas 成本。 某些类型针对没有写访问权限的存储进行了优化。

## **标头参数类型**

-   `time`: 消息创建时间戳。 用于重放攻击保护，以毫秒为单位编码为 64 位 Unix 时间。

> 规则: 合约应存储最后接受消息的时间戳。 初始时间戳为 0。 当收到新消息时，合约应做以下检查：

`last_time < new_time < now + interval, where`

`last_time` - 最后接受的消息时间戳（从 c4 寄存器加载），
`new_time` - 入站外部消息时间戳（从消息正文加载），
`now` - 当前区块创建时间（就像 NOW TVM 原语一样），
`interval` - 30 min.

如果满足这些要求，合约应继续执行。 否则，应拒绝入站消息。

-   `expire`: Unix 时间（以秒为单位，32 bit）之后合约将不处理消息。 它表示丢失的外部入站消息。

>规则：如果合约执行时间小于`expire`时间，则继续执行。 否则，消息会过期，并且事务会自行中止（通过`ACCEPT`原语）。 客户端等待消息处理直到`expire`时间。 如果在该时间间隔内未处理该消息，则认为该消息已过期

-   `pubkey`: 来自用于签署消息正文的密钥对的公钥。 该参数是可选的。 客户端决定是否需要设置公钥。 如果提供参数，则将其编码为位 1 后跟 256 位公钥，否则编码为位 0。
-   标头还可能包含自定义检查使用的任何标准 ABI 类型。

**函数参数类型：**

-   `uint<M>`: 无符号 M 位整数。 `Big-endian` 编码的无符号整数存储在 `cell-data` 中。
-   `int<M>`: 二进制补码有符号 M 位整数。 存储在单元数据中的大端编码有符号整数。
-   `bool`: 相当于 uint1。
-   tuple `(T1, T2, ..., Tn)`: 包含的元组 `T1, ..., Tn, n>=0` 以下列方式编码的类型：

```
Enc(X(1)) Enc(X(2)) . . ., Enc(X(n)); where X(i) is value of T(i) for i in 1..n
```

元组元素被编码为独立值，用于存储在不同的单元格中

`T[]` 是一个由 `T` 类型元素组成的动态数组。 它被编码为 TVM 字典。 `uint32` 定义放置在单元格主体中的数组元素计数。 `HashmapE`（参见 TVM 规范中的 TL-B 模式）结构随后被添加（一位作为字典根，如果字典不为空，一个数据引用）。 字典键是数组元素的序列化`uint32`索引，值是序列化的数组元素，类型为`T`。

-   `T[k]` is a static size array of `T` type elements. Encoding is equivalent to `T[]`
-   `bytes`: an array of `uint8` type elements. A separate cell holds the array. In the case of array overflow, the maximum cell-data size it's split into multiple sequential cells.
-   Note: contract stores this type as-is without parsing. For high-speed decoding, cut reference from body slice as `LDREF`. This type helps store some raw data in the contract without write or random access to elements.


> 注意：Solidity 中`bytes` 的模拟。 在 C lang 中可以用作 `void*`。

- `fixedbytes<M>`：`M``uint8` 类型元素的固定大小数组。 编码等价于 `bytes`。
- `map(K, V)` 是带有 `K` 类型键的 `V` 类型值的字典。 `K` 可以是任何 `int<M>/uint<M>` 类型，其中 `M` 从 `1` 到 `1023`。 字典被编码为“HashmapE”类型（如果字典不为空，则将一位作为字典根放入单元数据中，另一位与参考数据一起放入）。
- `address` 是 TON 区块链中的账户地址。 编码为 `MsgAddress` 结构（参见 TON 区块链规范中的 TL-B 模式）。
- `cell`：一种用于定义原始细胞树的类型。 存储为当前单元格中的引用。 必须使用“LDREF”命令解码并按原样存储。
- 注意：这种类型对于以“消息”结构的“StateInit”结构形式将有效载荷存储为类似于合同代码和数据的单元树非常有用。


## **单元数据溢出**

如果参数数据不适合当前单元格数据的可用空间，它将移动到一个单独的新单元格。 此单元格附加到当前单元格作为参考。 然后新单元格成为当前单元格。


## **单元格引用限制溢出**

为简单起见，此 ABI 版本为单元格数据溢出保留了最后一个单元格参考点。 如果当前单元格中的单元格引用限制已经达到（保留点除外）并且需要一个新单元格，则认为当前单元格已完成生成一个新单元格。 保留点存储对新单元格的引用，并继续将新单元格作为当前单元格。

如果以下所有参数都适合当前单元格，则需要引用的参数序列化（`cell`、`bytes`、`map`、`array` 类型）可以使用最后一个单元格引用。

**合约接口规范**

一个名为 contract ABI 的 JSON 文件存储合约接口。 它包括所有具有 ABI 类型描述的数据的公共函数。 下面是一个 ABI 文件的结构：

```
{

       "ABI version": 2,

       "header": [

              ...

       ],

       "functions": [

              ...           

       ],

       "getters": [

        ...

       ],

       "events": [

              ...    

       ],

       "data": [

              ...

       ]

}
```

Getters 是可在本地 TVM 上调用的 get 方法列表。


## **标题部分**

本节描述合约内功能的附加参数。 特定于头的类型被指定为带有类型名称的字符串。 其他类型被指定为函数参数类型（见**函数部分**）

```
"header": [

       "header_type",

       {

              "name": "param_name",

              "type": "param_type"

       },

       ...

]
```

示例:
```
"header": [

       "time",

       "expire",

       {

              "name": "custom",

              "type": "int256"

       }

]
```


## **功能部分**

**Functions** 部分指定每个接口函数签名，包括其名称、输入和输出参数。 合约接口中指定的函数可以通过 ABI 调用从其他合约或区块链外部调用。

**功能**部分具有以下字段：

```
"functions": [

       {             

              "name": "method_name",

              "inputs": [{"name": "func_name", "type": "ABI_type"}, ..],

              "outputs": [...],

              "id": "0xXXXXXXXX", //optional

       },

       ...

]
```

- `name`：函数名；
- `inputs`：一个对象数组，每个对象包含：
- `name`：参数名称；
- `type`：规范参数类型。
- `components`：用于元组类型，可选。
- `id`：可以添加可选的 `uint32` `id` 参数。 这个 `id` 将用作 `Function ID` 而不是自动计算。 PS：最后一种情况可用于不兼容 ABI 的合约。
- `outputs`：类似于 `inputs` 的对象数组。 如果函数不返回任何内容，则可以省略；


## **活动事件**

本节指定合同中使用的事件。 事件是正文中带有 ABI 编码参数的外部出站消息。

```
"events": [

       {             

              "name": "event_name",

              "inputs": [...],

              "id": "0xXXXXXXXX", //optional

       },

       ...

]
```


`inputs` 具有与函数相同的格式。

  

## **数据部分**

本节介绍合约全局公共变量。

```
"data": [

       {             

              "name": "var_name",

              "type": "abi_type",

              "key": "<number>" // index of variable in contract data dictionary

       },

       ...

]
```


## **Getters 部分**

尚不支持 Getters 规范，因此忽略此部分。

  >  [原文链接](https://docs.ton.dev/86757ecb2/p/40ba94-abi-specification-v2)
