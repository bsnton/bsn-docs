###### SMART CONTRACT LORE

# 管理TON的Gas
## Gas 基础知识

### **规范概述**

TVM的整个状态由五个部分组成:
- 栈（stack）
- 控制寄存器（control registers）
- 当前延续（current continuation）
- 当前代码页（current codepage）
- gas限制（gas limits）
这些统称为SCCCG。
查看第 1.4 节 [TVM specification.](https://ton.org/tvm.pdf)

**Gas** 组件限制了 gas 的使用，并且包含四个有符号的 64-bit整数：
- 剩余gas：gr
- 当前gas限制：gl
- 最大gas限制：gm
- gas信用：gc

以下始终是正确的：
		**0 ≤ gl ≤ gm, gc ≥ 0, and gr ≤ gl + gc**

对于内部消息，**_gc_** 初始化为0,**_gr_**初始化为**_gl + gc_**，并随着**_TVM_**的运行逐渐减少。当**_gr_**变为负值或者**_contract_**在**_gc > 0_**终止时，会触发**_out of gas_**异常。
### Gas 价格

正如  [TVM specification:](https://ton.org/tvm.pdf)A.1 中所述。
根据最初的 TON，对于大多数基元，gas 是根据以下公式计算的：
**Pb := 10 + b**
其中 b 是以位为单位的指令长度。 TON Labs 的实施也是如此。

**例如**：A0(ADD)指令所需gas为10+8=18gas，而A6cc(ADDCONSTcc)指令所需gas为10+16=26gas。

对于某些说明，此规则不适用。 TVM 规范明确列出了总 gas 价格，或者除了基本 Pb 之外的价格。

可以在 TVM 规范的 A.2 到 A.13 中获得带有附加信息的指令列表。

除了整数常量外，还可能出现以下表达式：
- 加载单元的总价。 目前它是每个单元 100 个gas单位。 现在再次重新加载一个单元格需要 25 个gas单位。
- 从 Builders 创建新单元的总价。 目前是500个gas单位。
- 异常抛出。 每个例外 50 个gas单位。
- 每个隐式 RET 退出区块需要 5 个 gas 单位。 跳转到第一个链接需要花费 10 个gas单位 - 隐式 JUMP。
- 如果参数超过 32 个，则转移到新的继续传递参数会消耗 gas。 它花费 N-32 gas，其中 N 是参数的数量。
- 元组gas价格。 每个元组元素有 1 个gas单位。

    >**请注意**，最昂贵的操作是字典读/写操作。 字典以单元格树的形式存储，其中每个单元格只能与其他四个单元格链接。 因此，这些树可能会变得非常大，具体取决于需要存储的数据。 要读取任何单元格中的数据，需要首先读取其所有父单元格，以每个单元格 100 gas 的价格，并在单元格中写入数据，类似地，需要（重新）创建其所有父单元格的价格为 每个电池 500 gas。


### **全局gas限制**
全局gas限制是存储在主链配置合约中的值。 全局值是标准的，不会在合约部署时改变。 只有验证者共识才能修改它们。

-   当前使用的值始终可以在[ton.live](http://ton.live/) 上最新的关键块详细信息（示例）中查看， ([example](https://net.ton.live/blocks?section=details&id=8cee868a94b1e22794a927279286dc95498310cda982f4657e351a3da693cf27)). p20 配置参数值用于主链，p21 值用于工作链。
### **Gas-related TVM原语**

这些是用于 gas 相关操作的官方 TVM 原语列表：

- F800 — ACCEPT，将当前gas limit gl 设置为其最大允许值gm，并将gas credit gc 重置为零，在此过程中将gr 的值减少gc。换句话说，当前的智能合约同意购买一些 gas 来完成当前的交易。需要此操作来处理外部消息，这些消息本身没有带来任何价值（因此没有 gas）。
- F801 — SETGASLIMIT (g – )，将当前 gas 限制 gl 设置为 g 和 gm 的最小值，并将 gas 信用 gc 重置为零。如果到目前为止消耗的gas（包括当前指令）超过 gl 的结果值，则在设置新的气体限制之前抛出（未处理的）gas不足异常。请注意，带有参数 g ≥ 2 63 − 1 的 SETGASLIMIT 等效于 ACCEPT。
- F802 — BUYGAS (x – )，计算可以购买 x 个纳米代币的 gas 量，并以与 SETGASLIMIT 相同的方式相应地设置 gl。
- F804 — GRA​​MTOGAS (x – g)，计算可以用 x 纳米代币购买的气体量。如果 x 为负，则返回 0。如果 g 超过 2 63−1，则用此值替换。
- F805 — GASTOGRAM (g – x)，以纳米代币计算 g gas 的价格。
- F806–F80F — 保留用于与气体相关的原语。这些尚未发布。

**注意**：F802、F804、F805 未在 Telegram TON 节点中实现。

在 TON Labs Node 中，通用gas公式与 TON 规范中指定的相同。总体而言，TON Labs 节点的运行符合规范。

对于每个执行的原语，根据规范公式向虚拟机添加gas量。每个基元的 Gas 值都基于 **gr**。


### **Gas 初始类型**

**从合约调用合约**

收到带有余额值的内部消息。在这种情况下，将应用以下公式来确定限值：

-   gm = min(account 余额 / gas 价格, global_gas_limit)
-   gl = min(message 值 / gas 价格, global_gas_limit)
-   gc = 0
-   gr = gc + gl

默认情况下，gas 成本分配给通过消息触发交易的调用者合约。内部合同也可以接受。如果 ACCEPT 没有被调用，则根据消息值从调用者合约中获取 gas。换句话说，消息值定义了电流限制。消息值决定了起始 TVM  gas限制。

所以，说白了，如果 ACCEPT 没有被调用，消息支付，如果 ACCEPT 被使用，目标合约可以购买额外的 gas。这种方法可以实现灵活的合约设计，其中要么总gas由调用方合约支付（但在这种情况下，它必须在任何时候都有足够的gas）要么目标合约也产生成本。

**2. 链下合约调用**

外部消息不携带余额值。 在这种情况下，根据以下公式计算值：

-   gm = min(账户余额/gas 价格，global_gas_limit)
-   gl = 0
-   gc = min(gm, global_gas_credit)
-   gr = gc + gl


由于外部消息没有 gas 值，gas 被信任执行它。 目标合约必须通过调用 Accept 来购买gas来支付成本。

如果在给予信用之前合约返回异常，则不收取 gas 费用。

由于 node 的公共代码刚刚发布，此文档可能会更新。
## in Solidity管理gas

### 理论
任何人都可以向您的合约发送外部消息。当消息到达时，合约初始 gas 限额等于 10,000 单位信用 gas，稍后应由 ACCEPT TVM 原语购买。否则，当信用 gas 降为零时，TVM 会抛出 out of gas 异常。合约应该花费这 10,000 单位的“free”gas 来检查入站消息的正文 tp 确保它是有效的并且可以被合约成功处理。

信用gas津贴的想法是，只要它超过零，合约抛出的任何异常都会阻止所有进一步的gas费用。但是一旦合约接受消息，无论交易是否中止，合约消耗的所有gas都会转换为gas费用。

ACCEPT 在内部消息中也很有用。当另一个合约向您的合约发送内部消息时，初始 gas 限制等于入站消息值除以 gas_price 或全局 gas 限制（如果较小）。如果此值不足以完成执行，则合约可以通过调用 ACCEPT 或 SETGASLIMIT 原语来增加其 gas 限制。 ACCEPT 原语增加其余额值除以 gas_price 的限制，SETGASLIMIT 原语将当前 gas 限制设置为从 TVM 堆栈中弹出的值（该值不能大于 **gm** 限制）。

使用 ACCEPT 命令，合约可以选择是由调用者合约还是合约本身支付其执行的 gas。

### 执行
在 TON Labs 中，ACCEPT 原语在 Solidity 中实现为公共函数调用的私有函数。

在下面找到实际使用示例。 所有这些都可以使用 TON Labs Solidity 编译器进行编译。
### **Accept gas 函数*
```
 contract AcceptExample1 {
  uint _a = 0;
 ​
  function foo(uint a) public {
   require(a > 0, 100); //do checks
   tvm.accept();    //accept gas only after check
   _a = a;       //do other logic
  }
 }
```
为了避免在 foo 函数被另一个合约调用时支付 gas，我们可以使用以下代码：
```
function foo(uint a) public {
   require(a > 0, 100); //do checks
   if (msg.sender == 0) {
    //accept gas only in case of external message.   
    tvm.accept(); 
   }
   _a = a;      //do other logic
  }
```

**重要提示**：在从入站消息正文反序列化参数之前调用修饰符。 在上面的例子中，**_AlwaysAccept()_** 将在 **_a_** 和 **_b_** 解码之前被调用。
