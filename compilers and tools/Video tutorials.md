###### COMPILERS AND TOOLS

# Video 教程

[Solidity Compiler Live Guide](https://www.youtube.com/watch?v=JljPeTmB2nU)  
[Sol2TMV Optimization Progress](https://www.youtube.com/watch?v=I9cPZyt3pAc)  
[Solidity on TON: mappings and addresses](https://www.youtube.com/watch?v=uIJlR0aAZS8)  
## 参考案例

### 从容器中访问linker
如果您有 Node SE 但想单独使用编译器，请遵循以下指南。

要访问的实用程序:

1.  拉取编译器工具包: `docker pull tonlabs/compilers`
2.  从带有合约的文件夹中调用命令行（或终端）
3.  访问linker: `docker run -it --rm -v $(pwd):/projects -w /projects -e USER_AGREEMENT=yes tonlabs/compilers`

现在您可以使用TVM Linker CLI.
### 生成 .tvc 和 .boc

TVM-linker可以生成一个随时可以部署的合同, `.tvc`格式或 `.boc`格式的消息。

**.tvc文件**
要生成文件，请调用：
```
tvm_linker compile <source> [--lib library [...]] [--abi-json <abi_file>] [--genkey | --setkey <keyfile>]
```
其中:

`source` 代表 TVM 汇编器源文件的名称，编译后在您的项目文件夹中可用。

`library` 是运行时库文件名的列表。.

Linker生成 address.tvc 文件，其中 address 是来自初始数据和合约代码的哈希。

如果有合约ABI文件，最好使用--abi-json选项提供合约ABI文件。 然后根据 ABI 中的函数签名生成函数 ID。

要生成新的密钥对并将公钥放入合约，请调用：
```
tvm_linker compile <source> --genkey <key_file>
```
**.boc 信息**
使用此选项部署到 TON 测试网。

> 请注意，如果要将合约部署到 TON 测试网，还必须根据[官方文档](https://zeroheight.com/86757ecb2/p/142588).

1.  首先，如上所述生成可部署的合约。
2.  然后以文件名为参数调用`test`linker子命令，获取合约地址。
3.  然后调用`message`子命令，以生成的合约地址为参数，创建`.boc`格式的外部入站消息（测试文件的实际视图请参见下面的截图，还不够 调用 `tvm_linker message <contract-address>` 来获取一个文件）：
4. 
```
tvm_linker message <contract-address> [--init] [--data] [-w]
```
其中 contract-address 是编译后的合约文件 (.tvc) 的名称，没有扩展名。

如果要创建用于部署合约的构造函数消息，请使用 --init 选项：
```
tvm_linker message <contract-address> --init
```
然后使用从`address.tvc`文件加载的代码和数据创建一个新的 _constructor message_（即在网络中启动合约部署的消息）。
​![](https://docs.ton.dev/uploads/Vguv6PZLahyugKFmF0cN7A.png)

查看更多文件和样本，请访问[https://github.com/tonlabs/samples/tree/master/solidity](https://github.com/tonlabs/samples/tree/master/solidity)。

[原文链接](https://docs.ton.dev/86757ecb2/p/9504e8-video-tutorials)
