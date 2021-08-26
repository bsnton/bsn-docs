# TONDev

https://github.com/tonlabs/tondev
Free TON 开发环境
## 什么是TONDev？

[TONDev](https://github.com/tonlabs/tondev) 是一个带有 CLI 接口的 Node.js 包，它允许从开发人员的单个接口执行以下用例：
### 用例

-   轻松管理所有核心[Free TON 开发者工具](https://ton.dev/)
-   配置网络（包括本地区块链、开发者网络、Free TON(主)网络）：添加、配置Giver；
-   管理密钥：添加、删除
-   使用来自 CLI 的免费 TON 区块链

此外，该项目用作[TONDev VS Code 扩展](https://github.com/tonlabs/tondev-vscode)的后端。

### TONDEV支持哪些工具？

在第一次请求时为目标平台自动下载和安装组件。

-   [Solidity 编译器](https://github.com/tonlabs/TON-Solidity-Compiler)
    
-   [C/C++ 编译器](https://github.com/tonlabs/TON-Compiler)
    
-   [合约管理工具](https://github.com/tonlabs/tondev#contract-management)- 从 CLI 处理您的合约。使用方便的 CLI 命令部署和运行您的合约
    
-   [网络工具](https://github.com/tonlabs/tondev#network-tool)- 管理您的网络：添加、删除、配置给予者
    
-   [签名者工具](https://github.com/tonlabs/tondev#signer-tool)- 管理您的密钥和种子短语：创建您的秘密一次，并通过合约管理工具的别名使用它。真的很容易。
    
-   [TON OS Startup Edition](https://github.com/tonlabs/tonos-se)——用于开发和测试的本地区块链
    
-   [Debot browers](https://github.com/tonlabs/debots)- Web debot 浏览器。目前，支持 Extraton Debot 浏览器。即将支持 Surf Debot 浏览器。
    
-   [TestSuite4](https://github.com/tonlabs/TestSuite4) – 用于合约测试的 Python 轻量级框架。
    
-   [tonos-cli](https://github.com/tonlabs/tonos-cli) – 用于多重签名钱包管理和抵押的命令行工具，包括 CLI Debot 浏览器。TONDev 有助于安装和版本管理。
    

TONDev 可以按照[集成说明](https://github.com/tonlabs/tondev#tondev-extensibility)使用其他工具进行扩展。

## 快速开始
## 安装

### [](https://github.com/tonlabs/tondev#prerequisites)依赖

-   [`Node.js`](https://nodejs.org/)  >= 14.x installed
-   (选项)  [`Docker`](https://www.docker.com/)  >= 19.x installed
-   Solidity 编译器需要 Windows 上的 VC++ 运行时。您可以从[最新支持的 Visual C++ 下载](https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0)安装它。

### 安装
npm i -g tondev

### 更新
npm r -g tondev  
npm i -g tondev


## 命令行界面
### General command syntax
```
tondev <tool> <command> ...args
```



