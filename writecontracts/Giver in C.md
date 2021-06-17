WRITE SMART CONTRACTS

# 使用C写Giver

*本教程旨在向您介绍用 C 编写合约。它演示了如何编写一个简单的Giver合约和一个客户端。*

*假设您熟悉 C 语言语法和语义以及 Free TON 区块链基本设计原则。*


合约的完整源代码可在此处[here](https://github.com/tonlabs/samples/tree/master/c/example-8-gramgiver)获得。

## ABI

智能合约Smart Contract (SC) 开发从定义合约公共接口的 ABI 文件开始，并允许从另一个 SC 或链下应用程序进行异步函数调用。

ABI 可以与 ELF 文件的导出和导入表进行比较，但是它不包含在二进制文件中以降低存储成本。

ABI JSON 包含 4 个主要部分：

- ABI版本
- SC定义和使用的公共函数列表
- 可选的事件列表(当前不支持C;所以列表必须为空)
- 持久性数据的可选列表(过时的特性)

   >请注意，For SC in C ABI 文件应该手动编写，然后位于 `TON-Compiler` 存储库中 `/stdlib` 的工具 `abi_parser.py` 为 C 中的合同生成必要的样板代码。

每个合约都必须有一个构造函数，一个在合约部署到网络中时运行的函数。

让我们描述一个给定客户合同的 ABI (client.abi)：
```
{
	"ABI version": 1,
	"functions": [
		{
			"name": "requestGrams",
			"inputs": [
				{"name":"remoteContractAddress","type":"address"},
        {"name":"value", "type":"uint64"}
			],
			"outputs": []
		},
		{
			"name": "getGrams",
			"inputs": [
				{"name":"value","type":"uint64"}
			],
			"outputs": []
		},
		{
			"name": "constructor",
			"inputs": [],
			"outputs": []
		}
	],
	"events": [],
	"data": []
}
```
-   地址`remoteContractAddress`（`MsgAddressInt*` in C code）类型
-   `uint64`（unsigned in C code）类型的值
    >下面介绍了 ABI 类型与 C 类型的对应关系。
 
ABI 类型定义传入消息的解析方式。 所以，尽管 TON对C不区分 `char`、`short`、`int`、`long` 等类型，
**将传入值声明为 64-bits无符号值可缩短传入消息并降低处理成本（gas）**.

`requestGrams` 向部署在` remoteContractAddress` 的合约发送内部消息，该消息需要手动序列化，但是对于外部消息，程序员可能会定义输出列表并自动生成序列化逻辑。

该合约还使用了Giver合约中的 getGrams 函数。 请注意，当前版本的 ABI 不提供任何描述所有权的机制。 最后，合约具有构造函数，它必须存在于每个合约中，并且不能接受或返回任何值。

Giver本身的接口如下所示：
```
{
	"ABI version": 1,
	"functions": [
		{
			"name": "getGrams",
			"inputs": [{"name":"value","type":"uint64"}],
			"outputs": []
		},
		{
			"name": "constructor",
			"inputs": [],
			"outputs": []
		}
	],
	"events": [],
	"data": []
}
```
它只有` constructor` 和 `getGrams` 函数。
一旦描述了 ABI，我们就可以运行 `abi_parser.py` 以通过以下方式为每个合约生成包装器和标头：
 ````
 $ abi_parser.py giver
 $ abi_parser.py client
 ````   
 
请注意，*.abi 不能跟在合同名称之后。 头文件（giver.h）应该包含在相应的合约实现中，包装器（giver_wrapper.c）包含一个实现消息处理逻辑。
## 公共函数
`abi_parser.py` 为每个公共函数生成一个由 3 部分组成的代码：
1.  反序列化传入的消息正文。
2.  使用反序列化参数调用 `<function name>_Impl`。
3.  序列化函数结果（if not void）并将其作为外部消息发送。

在 `requestGrams` 的情况下，生成的代码如下所示：
```
void requestGrams () {
    MsgAddressInt remoteContractAddress_Deserialized = Deserialize_MsgAddressInt ();
    unsigned value_Deserialized = Deserialize_Unsigned (64);
    requestGrams_Impl (remoteContractAddress_Deserialized, value_Deserialized);
}
```

`requestGrams_Impl` 获取的参数必须具有与` requestGrams` 输入的 ABI 类型对应的类型，并且必须返回与 ABI 中输出值的类型对应的值。

**对应表:**
```
uint1 .. uint256 -> unsigned

int1 .. int257 -> int

address -> MsgAddressInt*

cell -> __tvm_cell (see below)
```

当前不支持包括指针在内的其他类型作为公共函数参数。
## 外部函数
外部函数需要模拟定义，我们建议定义模拟` <externalFunction>_Impl` 而不是编辑自动生成的函数体。

如果 `<externаlFunction>` 仅用作向 发送消息的公共方法，则合约将忽略此定义。 但是，链接器必须生成正确的 ID（请参阅消息结构）(see [Message structure](https://docs.ton.dev/86757ecb2/p/16892e-giver-in-c/t/157b12))。

在客户端合约中的`getGrams` 中，Impl 对应物的定义是微不足道的：
```
void getGrams_Impl(unsigned value) {}
```
## 消息
公共合约功能之间以及链下应用程序与合约之间的所有交互都是通过消息执行的。

在 C 中，消息发送是显式的，这意味着您不能只调用公共函数，必须使用 `send_raw_message`。 但是，可以以标准方式调用辅助合约函数。
### 接受
当链下应用程序通过外部消息调用合约并且合约打算处理它时，需要通过调用 `tvm_accept()` 来明确接受。
在调用` tvm_accept()` 之前，合约可能会执行一些廉价的检查，例如 发件人地址、签名等。 `tvm_accept()` 暗示合约同意支付处理传入消息的费用。 如果 `tvm_accept()` 没有被调用或合约在 `tvm_accept()` 之前终止（例如，因为它耗尽了初始检查的 10,000 个 gas 单位），则所有计算都将被丢弃。
在我们的示例中，两个构造函数都没有进行任何检查 `tvm_accept()` 消息。 它们应该由外部工具（例如`tonlabs-cl`）部署，因此` tvm_accept()` 是必须的。
### 架构
描述了一般消息结构。 [https://ton.org/tblkch.pdf](https://ton.org/tblkch.pdf).

然而，TON 实验室引入了额外的消息正文约定。 身体包括：
-   函数 ID 或地址（可以通过 (unsigned) &functionName 获得）
-   调用的序列化参数

`abi_parser.py` 自动生成消息反序列化（从传入消息到 C 基元类型的转换）和序列化（从函数返回的 C 结构到消息的转换）。

内部消息序列化要么是手写的，要么是通过宏生成的。 为了通过宏自动生成序列化，client.c 通过以下方式定义传出消息体：
```
#define TON_STRUCT_NAME Message_uint64
#define TON_STRUCT \
    FIELD_UNSIGNED(rpaddr, 32) \
    FIELD_UNSIGNED(value, 64)
#include "ton-sdk/define-ton-struct-header.inc"
#define TON_STRUCT_NAME Message_uint64
#define TON_STRUCT \
    FIELD_UNSIGNED(rpaddr, 32) \
    FIELD_UNSIGNED(value, 64)
#include "ton-sdk/define-ton-struct-c.inc"
```
`ton-sdk/define-ton-struct-header.inc` 定义结构本身，`ton-sdk/define-ton-struct-c.inc` 定义结构的序列化和反序列化方法。

这两个文件都位于 TON-Compiler 存储库中的 /stdlib/ton-sdk。

序列化方法：
-   `Serialize_<typename>(<typename>*)` 将 `Message_uint64` 结构存储到 `c6` 寄存器，
-   `Serialize_<typename>_Impl(__tvm_builder*, <typename>*)` 执行相同的操作，但针对用户定义的构建器而不是 `c6`。

反序列化方法使用类似的命名约定：
 `Deserialize_<typename>(<typename>)` 和 
  `Deserialize_<typename>_Impl(_tvm_slice*, <typename>*)`。
  
  程序员很少需要实际编写这些函数，而是在 `abi_parser.py` 生成的代码中调用它们。
  
  在我们的示例中，客户端通过调用（发送消息）Giver的 `getGrams` 函数来请求 [test] 硬币。
  
  我们来看一下对应的代码：
```
void build_internal_uint64_message (MsgAddressInt* dest, unsigned data) {
    build_internal_message_bounce(dest, 0, 0);
    Message_uint64 msg = {(unsigned) &getGrams, data};
    __tvm_builder builder = __builtin_tvm_cast_to_builder(__builtin_tvm_getglobal(6));
    Serialize_Message_uint64_Impl(&builder, &msg);
    __builtin_tvm_setglobal(6, __builtin_tvm_cast_from_builder(builder));
}
```
第一行为消息创建一个标题并将其保存到 `c6` 寄存器。 第三行从`c6` 寄存器中获取一个构建器（`__builtin_tvm_cast_to_builder` 是必要的，因为寄存器可能包含任何类型的数据）。 第四行将消息正文与消息头相加。 第五行将其存储到 `c6`。

发送消息：
```
send_raw_message (MSG_PAY_FEES_SEPARATELY);
```
MSG_PAY_FEES_SEPARATELY 在这里意味着发件人自己为它的消息付费。

## 类型
C 语言使用 257-bits宽的有符号和无符号数字，因此：
```
sizeof(char) == sizeof(short) == sizeof(int) == sizeof(long) == sizeof(long long) == 1 byte == 257 bits
```
引入了 3 种额外的类型来处理 TVM 实体：

-   `__tvm_cell` 代表区块链中的通用存储单元；
-   `__tvm_builder` 将数据存储到单元格中；
-   `__tvm_slice` 从单元加载数据。

但是，在某些情况下，数据类型是未知的。 然后编译器假定它是一个整数：`cN` 寄存器，可通过 `__builtin_tvm_getglobal(unsigned)` 访问，因此有内置函数可以将整数转换为 TVM 类型：`__builtin_tvm_cast_to_<type>`，反之亦然：`__builtin_tvm_cast_from_<type>`。

## 持久数据和内存
SC 可以使用两种类型的内存：持久性和非持久性。 持久变量都是带有 `_persistent` 后缀的全局变量（例如来自 `client.c` 的 `numCalls_persistent`）。 它们驻留在区块链中，并且它们的值在合约运行之间保持不变（即持久）。 它很昂贵，但有时是必要的。

非持久性全局变量和一般的非持久性内存在其实现中使用类似的机制，因此建议尽可能避免使用非持久性内存。
## 工作流程
最后，我们总结了coin giver及其client的工作流程。

1. **Deploy.** giver和client都部署到区块链上。调用两个合约的 Сonstructor 函数并接受传入消息以成功部署。假设部署地址具有非零余额。
2. **用户（client）请求**。用户通过指定提供者地址和所需的nanocoins数量来调用“client”合约的“requestGrams”函数。
3. **请求处理和执行**。
4.`requestGrams`函数反序列化传入的外部消息，加载`giver`地址和请求的纳米币数量。
4. 它使用上述参数调用`requestGrams_Impl`。
5. `requestGrams_Impl` 增加 `numCalls_persistent` 值，使其与自 SC 部署以来对方法的实际调用次数相对应。
6. `requestGrams_Impl` 接受消息以确认即将进行的计算将被支付，因此不会被丢弃。
7. `requestGrams_Impl` 将公共消息头序列化为 `c6`
8. `requestGrams_Impl` 将包含 `getGrams` 函数的 ID 和请求数量的消息体序列化为 `c6`。
9. `requestGrams_Impl` 发送存储在 `c6` 中的内部消息。
10. `giver` 接收传入的消息，反序列化头部，反序列化主体中的函数 ID。
12.它通过ID找到`getGrams`函数并调用它。
11. `getGrams` 反序列化 value 参数并调用 `getGrams_Impl` 函数。
12. `getGrams_Impl` 接受传入的消息（对于内部消息是可选的）。
13. `getGrams_Impl` 使用请求的值创建一个空消息并将其发回。

 - [ ] [英文链接](https://docs.ton.dev/86757ecb2/p/16892e-giver-in-c)
