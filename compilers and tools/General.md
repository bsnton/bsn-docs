######  COMPILERS AND TOOLS
# 概要
## 工具
TON Labs 编译器工具链包括以下开源组件:

-   [Solidity to TVM Compiler](https://github.com/tonlabs/TON-Solidity-Compiler) 将Solidity中的智能合约编译成TVM组件
-   [LLVM-based Compiler](https://github.com/tonlabs/TON-Compiler) 将c++和C编译成TVM汇编
-   [TVM Linker](https://github.com/tonlabs/TVM-linker/tree/master/tvm_linker) 将TVM程序集转换为TVM字节码，链接到标准库，并将契约代码存储在二进制TVC文件中
-   [TON OS](https://github.com/tonlabs/tonos-cli) 用于生成地址、部署、查询和调用契约的实用程序
-   [TestSuite4](https://docs.ton.dev/86757ecb2/v/0/p/214408-testsuite4) 是一个用于简化TON合同开发和测试的框架。它包含轻量级区块链仿真器，可以很容易地以TDD-friendly的风格开发合约

## 安装
Solidity和C++编译器都可以在开源中获得。在相应的存储库中查看README文件:

[https://github.com/tonlabs/TON-Solidity-Compiler](https://github.com/tonlabs/TON-Solidity-Compiler)

[https://github.com/tonlabs/TON-Compiler](https://github.com/tonlabs/TON-Compiler)

用例 [https://github.com/tonlabs/samples](https://github.com/tonlabs/samples)。

## TVM Linker CLI
*阅读TVM_Linker命令行选项部分。*
### 关键用例
_tvm_linker_ 可用于:
-   **Linking contract binary code**. 在这种模式下，该实用程序将多个模块链接在一起，并提供最终的二进制文件（.tvc 文件），以便使用 TVM 进一步运行。
-   **Test run**. 在这种模式下，linker运行之前准备好的编译合约代码，模拟 TVM 的执行。 这可用于检查智能合约逻辑的正确性without deploying it to the blockchain.
-   **Message preparation**. 在此模式下，linker为 _.boc_ 格式的合同准备入站消息。

**Linking**
应使用以下命令行格式将多个模块链接在一起：
```
tvm_linker compile [--debug] [--lib <STDLIB>] <ASM_FILE> ...
```
参数:
-   `_-- <ASM_FILE>_`汇编程序文件的路径； 可以有多个汇编文件用于链接；
-   `_-- <STDLIB>_`标准库汇编文件的路径（例如，stdlib_c.tvm）；
-   `--debug` 启用链接的调试输出； 例如，它可用于获取全局符号值（例如，函数 ID）。

**测试运行**
将汇编程序文件链接在一起后，您可以使用linker运行已编译的合约。 该命令不启动节点，使用范围有限。

主要，测试运行用于代码部分的本地调试，不需要智能合约之间的交互（例如：单元测试）。

有关详细信息，请查看 [4) Emulating contract execution](https://github.com/tonlabs/TVM-linker/blob/master/README.md#4-emulating-contract-execution) 

**消息创建**

消息可以以 _.boc_ 格式发送到合约。 这些消息使用合约 ABI 和输入key-value对列表。

详细查看 [3) Preparing an external inbound messages in .boc format](https://github.com/tonlabs/TVM-linker/blob/master/README.md#3-preparing-an-external-inbound-messages-in-boc-format) 
