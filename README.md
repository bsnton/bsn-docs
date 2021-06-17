# TON OPB文档

## 大同链（基于Free TON）简介
“大同”链（基于Free TON）是在BSN环境中部署的基于智能合约的开放许可链平台。大同链基于对FreeTON/TON 进行改造，既利用了公链成熟的底层技术框架，又满足了中国市场的监管要求。提供对节点部署的许可控制，并取消使用 Token 支付 GAS（能量值） 的机制，能更好的满足中小企业以更具成本效益的方式快速开发和部署 DApp 的需求。  

大同链（基于Free TON）支持使用TVM(TON VM编译器)来执行用户编写的智能合约，同时支持Solidity、C、C++三种语言智能合约的编写和编译，并提供了多种IDE环境，便于开发者快速地搭建智能合约开发环境，进行合约开发、编译、调试、测试和发布部署。  

## 大同链（基于Free TON）核心技术优势：
* 快速、安全、可扩展，能够每秒处理数百万笔交易；
* 权益证明共识(POS)  和 拜占庭容错权益证明共识(BFTG)；
* 支持“任意”（图灵完备）智能合约；
* 同构/异构-混合多链架构，使系统具有高可扩展性；
* 支持动态分片，如果需要时自动创建额外的分片；
* 紧密耦合，可在所有区块链之间提供快速消息传递。

## 大同链（基于 Free TON）的基因组

如果想要构建一个可扩展的区块链系统，就必须从一开始就仔细选择其基因组。如果系统打算在未来支持一些在部署时未知的附加的特定功能，那么它应该从一开始就支持“异构”工作链（可能不同的规则）。要使系统真正具有可扩展性，它必须从一开始就支持分片；分片只有在系统“紧密耦合”时才有意义，因此这反过来意味着存在主链、快速的区块链间消息传递系统、BFT PoS 的使用等等。

当考虑到所有这些含义时，大同链（基于 Free TON）项目所做的设计选择都显得很自然，而且几乎是唯一可能的选择。

## DeBot介绍 - 大同链独有技术架构
DeBot（去中心化机器人）是 TON 区块链上智能合约的直观、无需先验知识的界面。

区块链技术很复杂，对于没有该领域经验或技术背景的用户来说很难学习。通过 DeBots，我们旨在简化在区块链上实现用户目标所需的交互，并简化基于区块链的服务的开发过程，同时保持此类产品的预期安全级别。

DeBot 最基本的是一个基于聊天的安全界面，它允许用户与区块链上的智能合约进行交互，并以对话的形式访问其各种功能。  

![](https://docs.ton.dev/uploads/KPwlh26fN52e0TF6YcpZSQ.svg)

Debot开发指南：https://github.com/tonlabs/debots

# WRITE目录
## 写智能合约
>[开始使用TON智能合约](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Getting%20started%20with%20TON%20smart%20contracts.md)
 >[使用Solidity写智能合约](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Write%20smart%20contract%20in%20Solidity.md)
> [使用C++写智能合约](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/C++%20Tutorial.md)
> [使用C语言写giver](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Giver%20in%20C.md)
> [TON solidity API](https://github.com/bsnton/bsn-docs/blob/9792d6a1a819fb04977380747908b06f6b5d0de8/write%20smart%20contracts/Solidity%20API%20for%20TON.md)
## 编译器和工具
>[概述](https://github.com/bsnton/bsn-docs/blob/911e94d2a92f390829eb6ed9dc3a863ab6443ec5/compilers%20and%20tools/General.md)
