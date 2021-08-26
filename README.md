# TON OPB文档

## 大同链（基于FreeTON）简介
大同链（基于FreeTON）是在BSN环境中部署的基于FreeTON技术框架的开放许可链平台。大同链基于对FreeTON进行改造，既利用了公链成熟的底层技术框架，又满足了中国市场的监管要求。提供对节点部署的许可控制，并取消使用 Token 支付 GAS（能量值） 的机制，能更好的满足中小企业以更具成本效益的方式快速开发和部署DApp的需求。  

大同链（基于FreeTON）是一个真正去中心化、分布式、安全、可持续、可扩展、低延迟的世界计算机，我们可以将其理解为一个超级分布式处理器(TON Kernel)的平台。

支持使用TVM(TON VM虚拟机)来执行用户编写的智能合约;支持Solidity、C、C++三种语言智能合约的编写和编译;同时支持13种主流语言的SDK开发,并提供了多种IDE开发环境,便于开发者快速地搭建智能合约开发环境，进行合约开发、编译、调试、测试和部署发布。用户可以使用GraphQL协议对链上数据进行全方位、快捷、灵活的检索。
![image](https://user-images.githubusercontent.com/85273157/130897662-5bf351a7-cb2a-4794-a00c-3fb131531993.png)




## 大同链(基于FreeTON)核心技术优势：
* 快速-能够每秒处理数百万笔交易；
* 安全-权益证明共识(POS)和拜占庭容错权益证明共识(BFT)；
* 高灵活性-支持图灵完备的智能合约；
* 高扩展性-同构/异构混合多链架构；
* 多链架构-由主链、工作链、分片链组成；主链可创建2^32个工作链、每个工作链最多可以分256个分片链，可以想象为区块链中的区块链；
* 无限分片-可以动态创建与合并分片(分片链)；
* 紧密耦合-可在所有区块链之间提供快速消息传递；
* 完全去中心化-全球400+个验证器。


## 大同链（基于FreeTON）的基因组

 如果想要构建一个可扩展的区块链系统，就必须从一开始就仔细选择其基因组。如果系统设想未来支持部署Dapp时未知的附加特定功能，那么它应该从一开始就支持“异构”工作链（可能不同的规则）。要使系统真正具有可扩展性，它必须从一开始就支持分片；分片只有在系统“紧密耦合”时才有意义，因此这反过来意味着存在主链、快速的区块链间消息传递系统、BFT、PoS的使用等等。
 当考虑到所有这些含义时，大同链（基于 FreeTON）项目所做的设计选择都显得很自然，而且几乎是唯一可能的选择。

## DeBot介绍 - 大同链独有技术架构
DeBot（去中心化机器人）是 TON 区块链上智能合约的直观、无需先验知识的界面。

区块链技术很复杂，对于没有该领域经验或技术背景的用户来说很难学习。通过 DeBots，我们旨在简化在区块链上实现用户目标所需的交互，并简化基于区块链的服务的开发过程，同时保持此类产品的预期安全级别。

DeBot 最基本的是一个基于聊天的安全界面，它允许用户与区块链上的智能合约进行交互，并以对话的形式访问其各种功能。  

![](https://docs.ton.dev/uploads/KPwlh26fN52e0TF6YcpZSQ.svg)

Debot开发指南：https://github.com/tonlabs/debots

# 写智能合约
## 写智能合约
>[使用TON智能合约说明](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Getting%20started%20with%20TON%20smart%20contracts.md)  
 >[使用Solidity写智能合约](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Write%20smart%20contract%20in%20Solidity.md)  
> [使用C++写智能合约](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/C++%20Tutorial.md)  
> [使用C语言写giver](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Giver%20in%20C.md)  
> [TON solidity API](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Solidity%20API%20for%20TON.md)
## 编译器和工具
>[概述](https://github.com/bsnton/bsn-docs/blob/911e94d2a92f390829eb6ed9dc3a863ab6443ec5/compilers%20and%20tools/General.md)  
>[TestSuite4](https://github.com/bsnton/bsn-docs/blob/f017d49b4832cf64f25e0bc2b1210d5386e24311/compilers%20and%20tools/TestSuite4.md)  
>[Video 教程](https://github.com/bsnton/bsn-docs/blob/5afe8c379c68b6cfe8f6f31b9458f2ce584b285a/compilers%20and%20tools/Video%20tutorials.md)    
>[C++ for TVM](https://github.com/bsnton/bsn-docs/blob/5571f1793475c5d055362b6ea9ec1d160cc5a3d4/compilers%20and%20tools/C++%20for%20TVM.md)  
>[C for TVM.md](https://github.com/bsnton/bsn-docs/blob/04427a0d25f835dda9710d7be80df37fc618c505/compilers%20and%20tools/C%20for%20TVM.md)

## TON智能合约知识  
>[概述](https://github.com/bsnton/bsn-docs/blob/5c7f47b2af82cd5706c36b6019f56312dddef25b/smart%20contract%20lore/Overview.md)  
>[管理TON的gas](https://github.com/bsnton/bsn-docs/blob/f16e66cf3b40980d652ffe973f14e2fcfddadb46/smart%20contract%20lore/Managing%20gas%20in%20TON.md)  
>[fee计算详情](https://github.com/bsnton/bsn-docs/blob/5c7f47b2af82cd5706c36b6019f56312dddef25b/smart%20contract%20lore/Fee%20calculation%20details.md)  
>[ABI 规范 V2](https://github.com/bsnton/bsn-docs/blob/06598e7a851b705c7825efa3190d92660357a3dd/smart%20contract%20lore/ABI%20Specification%20V2.md)  
>[安全](https://github.com/bsnton/bsn-docs/blob/06598e7a851b705c7825efa3190d92660357a3dd/smart%20contract%20lore/Security.md)  

## TONDev开发环境
>[TONDev](https://github.com/bsnton/bsn-docs/blob/5cd15ae06672d477f6a38263fe561abe31ee32a0/tondev/README.MD)
