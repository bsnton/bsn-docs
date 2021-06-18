###### COMPILERS AND TOOLS

# C for TVM
## C for TVM Library

_更新待定，可能已过时_

### 一般程序结构
传统的 C 程序只有一个 main 函数，它依次调用其他函数。 合约看起来更像是具有多种功能的类，可以从外部调用。 此外，为了更加相似，每个合约都有自己的内存（在 TVM 中调用持久性）。

因此，C 语言中的典型合约没有 main 函数，但有几个公共方法和几个持久变量。

### 公共方法

如果函数在 **X.abi**文件中声明，则该函数将成为合约 X 的公共方法。
### 全局和持久变量

C语言编写的程序调用一次代码：在代码开始工作之前，所有数据都被初始化； 执行结束后，所有数据都被处理掉。 就合约而言，它们在合约方法调用之间保留其状态。 所以有两种不同类型的全局变量：

-   传统的全局 C 变量在方法停止工作后被遗忘；
-   存储在区块链中的持久变量； 这些值在方法调用之间保持（保持）。

可以通过添加 _persistent 后缀来使全局变量持久化：
```
int x = 0;             // Forgotten after each method invocation
int y_persistent = 0;  // Kept in blockchain and does not lose its value
```
### 低级实现细节

上述原语是由 TON Labs 团队编写的，TVM 并没有真正强加这种通信格式。 事实上，合约只有一个入口点。 当发出请求（消息）时，会同时提供一个切片（即a bit string）。

TON 标准库的低级部分包含一个 **selector**：一个解析切片并调用给定合约方法的特殊函数。 方法选择由切片的初始字节中指定的方法 id 确定。 剩余的切片元素存储到工作切片中。

合约方法由特殊脚本根据 abi 文件生成。 然后将它们放入 **X_wrapper.c** 文件中，其中 X 是合约的名称。 这些方法从切片中反序列化参数并将它们作为参数传输到适当的合约函数实现。

### 结构库

该库是一组必须直接或间接包含到您的合约代码中以实现预定义 C 结构的文件。这些结构旨在确保更顺畅地开发可编译为 TVM 程序集的代码。

出于学习目的，我们提供了一个带注释的示例合同，展示了库文件如何包含在其中、它们如何工作以及如何编写与之相关的代码。该合约是为 Piggy Bank dapp 设计的区块链转账交易。

请注意，该库具有交叉引用结构。换句话说，包含在最终智能合约中的文件（带有声明的 .h 头文件）反过来又包含来自库的其他文件（带有定义的 .inc）。

此外，还有包含声明和定义实现的 .inc 或 .c 文件。但是，这些不包含在其他文件或合同本身中。

文件命名约定表明存在实现与 TON 兼容的 C 结构、消息传递逻辑、合同数据（字段）和允许在 C 中定义 TVM 特定组件的文件的文件。

**arguments.h** 文件可以在不引入额外结构的情况下实现更快的函数参数解析。

### 包含模式

**详细文件说明**


示例智能合约包括几个头文件。 基本上，它们允许创建智能合约的有效版本。

鉴于后续库的修订和修改，以及项目的特殊性，您可能会发现某些文件对您来说是多余的。

```
#include "ton-sdk/tvm.h"
#include "ton-sdk/messages.h"
#include "ton-sdk/arguments.h"
#include "ton-sdk/smart-contract-info.h"
```
合约的主体包含解析的参数、存储在永久内存中的业务逻辑和最终消息。 消息的第一部分取自 ABI 协议，第二部分由合约生成（大整数，46 个符号）。

**TVM.h**

**tvm.h** 文件声明了通用的 C 结构。
**注意**：值操作的函数体在 **tvm.c** 文件中实现，该文件与源文件一起编译并链接到生成的合约二进制文件。

  

**smart-contract-info.h, smart-contract-info.inc, define-ton-structure-header.inc**

**smart-contract-info.h** 头文件定义了 **SmartContractInfo TON** 特定的结构。 结构体的字段在 **smart-contract-info.inc** 文件中声明。 文件**define-ton-structure-header.inc** 包含一些辅助宏，它们声明：
- **SmartContractInfo**结构本身
- 生成器和单元序列化函数
- 切片和单元格反序列化函数
  >**注意**：请参阅 TVM 规范，了解有关单元袋、单元序列化和切片反序列化概念的更多信息。


**smart-contract-info.inc**在 TON_STRUCT 宏中定义了合约字段结构。 该宏列出了受支持的字段的名称、长度和类型。
```
#define TON_STRUCT_NAME SmartContractInfo
#define TON_STRUCT \\
    FIELD_CONSTANT_UNSIGNED(smc_info, 0x076ef1ea, 32) \\
    FIELD_UNSIGNED(actions, 16) \\
    FIELD_UNSIGNED(msgs_sent, 16) \\
    FIELD_UNSIGNED(unixtime, 32) \\
    FIELD_UNSIGNED(block_lt, 64) \\
    FIELD_UNSIGNED(trans_lt, 64) \\
    FIELD_UNSIGNED(rand_seed, 256) \\
    FIELD_COMPLEX(balance_remaining, CurrencyCollection) \\
    FIELD_COMPLEX(myself, MsgAddressInt)
#include HEADER_OR_C
#undef HEADER_OR_C
```

下面是对上面代码中的字段声明和宏的解释：

- 第一行定义结构名称，**SmartContractInfo**（该宏用于许多需要结构名称的地方）。
- **FIELD_CONSTANT_UNSIGNED** 宏定义了一个常量（“虚拟”）字段，该字段并未真正存储在结构中，但其值在序列化时放入切片并在反序列化时检查。
- **FIELD_UNSIGNED** 不需要深入的解释。
- **FIELD_COMPLEX** 允许定义嵌套结构。
- **HEADER_OR_C** 宏根据上下文包括源文件或头文件。 此宏在使用后未定义，以允许在没有警告已声明宏的情况下声明文件中的下一个结构（如果存在）。

这个复杂的包含和宏系统定义了许多有用的函数和结构。 例如，合约中的以下函数根据宏中定义的结构返回余额信息。

```
SmartContractInfo sc_info = get_SmartContractInfo();
int balance = sc_info.balance_remaining.grams.amount;
```


**message.h** 和 **message.inc**

**message.h** 文件声明了一个宏来启用消息传递过程。 In 指的是定义实际消息传递规则的 **message.inc** 文件。

**message.inc** 将消息传递规则映射到 **tvm.h** 和 **define-ton-struct-header.inc** 文件中声明的结构和字段。

上面介绍了**arguments.h** 文件。

### ABI注释

TON Labs 合约中的合约消息根据 ABI 规范进行格式化。 目前仅支持 (u)int 值。

  

## C 编译和部署

阅读下面的文档或转到教程：


### 语法差异

契约类似于 C 中的标准程序，但存在重要的语法差异：

  

1. ABI 文件。 ABI 文件对公共函数的签名和属性进行编码。此信息不存储在区块链中，而是以 JSON 格式提供，以实现最初以不同语言编写的合约之间的交互。 `abi_parser`生成编译所需的头文件和源代码文件，部署合约需要 ABI 文件本身。
2. 公共方法不得接受参数或产生结果。相反，它反序列化传入消息中的参数，然后序列化输出值以发送消息。 `abi_parser` 为 abi 的每个公共方法生成样板函数。这些函数执行序列化，调用相应的 `*_Impl`函数（例如` transfer_Impl `进行传输），并将其结果序列化。
3. 有持久性和非持久性全局数据。持久化数据标有 `_persistent `后缀（我们将来会使用 GCC 风格的属性）。在公共合约的函数调用之间保留持久值。

### 编译

要编译合约，请获取从二进制文件构建或随 Node SE 分发版交付的 TVM 的 Clang（请参阅 https://github.com/tonlabs/TON-Compiler）。

除了编译器之外，您还需要一个 ABI 解析器工具、C 运行时、SDK 头文件和源代码。 它们都位于存储库或分发包的 `stdlib` 子目录中：

-   ABI parser - `stdlib/abi_parser.py`
-   C runtime - `stdlib/stdlib_c.tvm`
-   SDK headers and sources - `stdlib/ton-sdk`

为了演示编译工作流程，我们使用来自编译器存储库 (`samples/sdk-prototype/piggybank.c`) 的示例存钱罐合约。

1.  运行 `abi_parser`:
```
 abi_parser.py path/to/piggybank/piggybank
```
**注意** .abi 扩展名不是强制性的。

该脚本生成 `piggybank.h`、`piggybank_wrapper.c`、`piggybank.err` 文件。

如果编译成功结束，最后一个文件必须为空。  


`piggybank.h`, `piggybank_wrapper.c` 包含公共函数的样板代码。

2. 运行编译:
```
 clang -target tvm -O3 -S piggybank.c piggybank_wrapper.c -I/path/to/stdlib
```  


   >**请注意**，现在用于 TVM 的 Clang 无法生成对象或 .boc 二进制文件，因此 -S 标志是生成程序集输出所必需的。

-O3 是我们在测试中也使用的推荐优化级别。 




3. 除了合约本身，编译 SDK 源代码：

```
 clang -target tvm -O3 -S /path/to/stdlib/ton-sdk/*.c -I/path/to/stdlib
```

4. 连接链接器的输出汇编文件（此实用程序现在不支持多文件输入）：

```
cat *.s > piggybank.combined.s
```
另一种选择是使用 `llvm-linker` 工具在 `LLVM IR` 级别链接模块。

5. 要链接合约，您需要 `piggybank.combined.s` 和 `piggybank.abi`：

```
tvm_linker compile piggybank.combined.s --abi-json piggybank.abi --lib /path/to/stdlib/stdlib_c.tvm
```

### 部署

部署步骤与语言无关。 请参考这里的相关文档 [https://github.com/tonlabs/samples/tree/master/solidity](https://github.com/tonlabs/samples/tree/master/solidity)，在 Node SE 文档或在 TVM 链接器 CLI 指南中。

  

### 示例教程

## Debugging

### 分析 & 测试


暂时没有办法分析合约返回值：合约发送外部消息，解析它们的工具还有待开发。

尽管如此，有两种方法可以将合约执行结果可视化：查看持久内存和将资金转移到其他账户。

对于以下问题，我们认为您已经将所有消息发送到合约，现在正在分析区块链的新状态。
  

### 持久内存

此方法要求您的合约将值写入持久内存。

1. 打开终端窗口。
2. 将您的目录更改为 <project-name>/build_contract_<contract-name>.c 。
3. 启动test-lite-client: **_test-lite-client -С ton-global.json._**
4. 使用 **_getaccount_** **_0:<account address>_** 命令检索联系人数据（期望一长串值）。
5. 找到 '**data:**' 字符串。 它包含在字符串后指定的持久内存的原始内容。
6. 完成后关闭终端。

### 转移资金


此方法要求您的合同将资金发送到另一个帐户。

1. 打开一个新的终端窗口。
2. 将您的目录更改为 `<project-name>/build_contract_<contract-name>.c`
3.启动test-lite-client：`**_test-lite-client -С ton-global.json._**`
3. 使用`**_getaccount_** **_0:<another account address_**`**_>_** 命令来检索联系人数据（需要一长串值）。
5.找到`**_storage/balance/grams/amount_**`字段（靠近账户信息的顶部）并检查其值。
4. 完成后关闭终端。

[原文链接](https://docs.ton.dev/86757ecb2/p/52da1e-c-for-tvm)
