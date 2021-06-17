# 使用Solidity写智能合约

## 在你开始之前
### 你需要
-   带有基本开发工具的 PC 或笔记本电脑
    -   (推荐: Ubuntu 18.04) Linux
    -   Windows
    -   MacOS
-   TVM汇编编译器的 Solidity
    -   (推荐) 构建TON-Solidity-Compiler[sources](https://github.com/tonlabs/TON-Solidity-Compiler)
    -   下载TON-Solidity-Compiler[binary](https://github.com/tonlabs/TON-Solidity-Compiler/releases/download/0.25/solc0.25.tar.gz)
    -   安装docker容器[container](https://hub.docker.com/r/tonlabs/compilers)
-   Solidity 中的合约代码
    -   使用下面的 Wallet.sol
    -   使用您自己的代码
    -   从示例中拉取一个示例 [repository](https://github.com/tonlabs/samples/tree/master/solidity)
-   将应用程序合约部署到区块链上
    -   构建TVM [sources](https://github.com/tonlabs/TVM-linker/tree/master/tvm_linker)
    -   下载一个TON-Solidity-Compiler [binary](https://github.com/tonlabs/TON-Solidity-Compiler/releases/download/0.25/tools0.25.tar.gz)
    -   下载一个docker容器 [container](https://hub.docker.com/r/tonlabs/compilers)
### 推荐设置
-   OS: Ubuntu 18.04是最容易运行.
    -   提示: 在VM中运行ubuntu正常，请查看 [install guide.](https://docs.ton.dev/86757ecb2/v/0/p/69f25e-get-ubuntu-vm/b/744d13)
-   从源码构建Solidity编译器 (4-6 minutes)
    -   从github checkout TON-Solidity-Compiler [github](https://github.com/tonlabs/TON-Solidity-Compiler/) (a few seconds);
    -   按要求安装依赖项 [README](https://github.com/tonlabs/TON-Solidity-Compiler/blob/master/README.md) (1-2 minutes)
    -   从源代码构建编译器 [README](https://github.com/tonlabs/TON-Solidity-Compiler/blob/master/README.md) (~3-5 minutes)
-   合约源代码:
    -   推荐使用Wallet.sol
-   命令行工具, 其中之一:
    -   下载二进制包 [binary](https://github.com/tonlabs/TON-Solidity-Compiler/releases/download/0.25/tools0.25.tar.gz) 
    -   从源码构建 [tvm_linker](https://github.com/tonlabs/TVM-linker/tree/master/tvm_linker) 和 [tonos-cli](https://github.com/tonlabs/tonos-cli) 

### 安装编译器

从开源代码安装TON Labs Solidity Compiler [repository](https://github.com/tonlabs/TON-Solidity-Compiler).
```
 git clone git@github.com:tonlabs/TON-Solidity-Compiler.git
 cd compiler 
 sh ./scripts/install_deps.sh
 mkdir build
 cd build
 cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF -DTESTS=OFF -DCMAKE_BUILD_TYPE=Debug
 make -j8
 ```  
 
 
### 获取合约源码
```
pragma solidity >= 0.6.0;

/// @title Simple wallet
/// @author Tonlabs
contract Wallet {
    // Modifier that allows function to accept external call only if it was signed
    // with contract owner's public key.
    modifier checkOwnerAndAccept {
        // Check that inbound message was signed with owner's public key.
        // Runtime function that obtains sender's public key.
        require(msg.pubkey() == tvm.pubkey(), 100);

        // Runtime function that allows contract to process inbound messages spending
        // its own resources (it's necessary if contract should process all inbound messages,
        // not only those that carry value with them).
        tvm.accept();
        _;
    }

    /*
     * Public functions
     */

    /// @dev Contract constructor.
    constructor() public checkOwnerAndAccept { }

    /// @dev Allows to transfer grams to the destination account.
    /// @param dest Transfer target address.
    /// @param value Nanograms value to transfer.
    /// @param bounce Flag that enables bounce message in case of target contract error.
    function sendTransaction(address payable dest, uint128 value, bool bounce) public view checkOwnerAndAccept {
         // Runtime function that allows to make a transfer with arbitrary settings.
        dest.transfer(value, bounce, 3);
    }
	
    // Function to receive plain transfers.
    receive() external payable {
    }
}
```  
  
 
 
 ### 编译
使用 Solidity Compiler 将合约代码编译为 TVM 汇编器。
```
<PATH_TO>/TON-Solidity-Compiler/compiler/build/solc/solc Wallet.sol
```
编译器生成 Wallet.code 和 Wallet.abi.json 以用于以下步骤。  
  
将标准库组装并链接到 TVM 字节码中：
```
<PATH_TO>/tvm_linker compile Wallet.code --lib <path-to>/TON-Solidity-Compiler/lib/stdlib_sol.tvm
```
你的合约的二进制代码记录在`<WalletAddress>.tvc`文件中，其中`<WalletAddress>`是合约的临时地址。  

### 部署
让我们将合约部署到 `net.ton.dev` 的 TON Labs 开发测试区块链上。
1) 确保 tonos-cli 在 $PATH 中：
```
export PATH=$PATH:<PATH_TO>/tonos-cli
tonos-cli config --url net.ton.dev
```
2) 为您的合约生成地址、密钥和种子短语(助记词)：
```
tonos-cli genaddr <WalletAddress>.tvc Wallet.abi.json --genkey Wallet.keys.json
```
您在区块链中的合约地址位于`Raw address:`之后

***重要提示：保存此值 - 您将需要它来部署您的合同并使用它。 我们将在下面将其称为`<YourAddress>`。 种子短语(助记词)也打印到标准输出。 密钥对将被生成并保存到文件 Wallet.keys.json 中。***

>请注意，在实际部署之前，您需要向该地址发送一些coins。 TON 部署合约是收费的，因此您的新合约将为此收费。

3) 获取一些 [test] coins到您的帐户。 选项是：
- 请朋友赞助您的合同部署；
- 从您的钱包账户转移一些货币；
- 在开发人员聊天中询问。

4) 检查预部署合约的状态。 它应该是`Uninit`：
``` tonos-cli account <YourAddress>```  

5) 使用以下命令将您的合约部署到所选网络（示例中为 TON Labs devnet）：
```
tonos-cli deploy --abi Wallet.abi.json --sign Wallet.keys.json <contract>.tvc {<constructor_arguments>}
```
*如果参数中省略了 --abi 或 --sign 选项，则必须在配置文件中指定。 见下文。*

6) 再次检查合约状态。 这一次，它应该是active。
  
7) 调用你的合约函数：
```
tonos-cli call '<YourAddress>' sendTransaction '{"dest":"DestAddress", "value":1000000000, "bounce":true}' --abi Wallet.abi.json --sign Wallet.keys.json
```  


## 进一步的步骤

现在您的合同已启动并正在运行！ 你可以：

-   查看 [Solidity API for TON](https://github.com/tonlabs/TON-Solidity-Compiler/blob/master/API.md)
-   查看更多 [contract samples](https://github.com/tonlabs/samples/tree/master/solidity)
-   深入研究TON智能合约开发的某些方面
-   从 [GitHub](https://github.com/tonlabs/tonos-cli) 中的源代码构建 CLI 实用程序以确保您拥有最新版本 
-   查看我们的研究论文和 [TON docs in readable format](https://docs.ton.dev/86757ecb2/p/07ddda-walk-through-the-catchain)

> 英文链接：[Write smart contract in Solidity](https://docs.ton.dev/86757ecb2/p/950f8a-write-smart-contract-in-solidity*)
