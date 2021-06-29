###### SMART CONTRACT LORE

# fee计算详情

## 介绍


交易费用由与执行单笔交易相关的几种不同费用组成。 交易本身是复杂的过程，费用是相对于执行它们的不同阶段支付的。

在本文档中，我们解释了如何计算费用。

我们将把 `transaction_fee` 定义为单笔交易的所有费用的总和。  

 `
transaction_fee = inbound_external_message_fee + storage_fees + gas_fees + total_action_fees + outbound_internal_messages_fee
`

Inbound_external_message_fee - 如果事务中导入了[入站外部消息](https://docs.ton.dev/86757ecb2/v/0/p/632251-fee-calculation-details/t/2519cd) ，则会扣除`Inbound_external_message_fee`.  

`storage_fees` - 自上次交易的那一刻起的[存储成本](https://docs.ton.dev/86757ecb2/v/0/p/632251-fee-calculation-details/t/01f655)(storage costs)。

`gas_fees` 包括与交易相关的所有gas费用。 您可以在 [Gas Calculation Basics](https://docs.ton.dev/86757ecb2/v/0/p/00f1c7-managing-gas-in-ton/t/9171b9)部分找到更多信息.

`total_action_fees` - 执行“发送消息”的[actions](https://docs.ton.dev/86757ecb2/v/0/p/632251-fee-calculation-details/t/10edd2)费用 

`outbound_internal_messages_fee`  计算为交易生成的所有[出站内部消息](https://docs.ton.dev/86757ecb2/v/0/p/632251-fee-calculation-details/t/78df9e) 的费用总和。

根据交易的性质，除仓储费外，所有这些都可能不适用。

下面我们详细研究这些类型的费用。

 
> 注意：不要将区块创建费用与本文档中讨论的费用混淆。 区块创建费是由选举人合约铸造并在验证者之间分配的新硬币，作为创建区块的奖励。 它不是交易费用的一部分。

## 存储费用Storage Fees
TON 中的每笔交易都有一个存储阶段，这意味着对帐户余额收取一定的存储费用。 此费用在交易之间的期间收取，并根据以下公式计算：
`
Storage fee = ceil ((account.bits * global_bit_price + account.cells * global_cell_price) * period / 2 ^ 16)
`

其中:

-   `account.bits` and `account.cells` 代表帐户结构中的许多位和单元格，表示为单元格树（包括代码和数据）。

-   `global_bit_price` 是一个全局配置参数（主链和工作链的 p18）：存储一bit的价格。
-   `global_cell_price` 另一个全局配置参数（主链和工作链的 p18）：存储一个单元格的价格。
-   `period` - 自上次存储费用支付以来的秒数。

 
   >**注意**:虽然 account.bits 通常很容易估计，但 account.cells 值对于不同类型的数据可能会有很大差异。 一个单元格可以包含不超过 1023 位和 4 个对其他单元格的引用。 合约代码和数值变量倾向于有效地打包到单元格中，从而导致大部分单元格是完整的，因此存储数据所需的单元格数量最少。 更复杂的数据结构可以以较低的效率打包到单元格中，占用更多的单元格来存储相同数量的数据。

**示例**：让我们计算在工作链上存储 1KB 数据一天的最低费用：

`global_bit_price` = 1
`global_cell_price` = 500
`period` = 86400 seconds
`account.bits` = 8192

`account.cells` value for 8192 bits of data is 9 (rounding 8192/1023 up to the nearest integer).
 account.cells最小值为8192 bits数据，约 9（将 8192/1023 整数）。
因此，最低仓储费的计算方法如下：
`
Storage fee = ceil((8192 * 1 + 9 * 500) * 86400 / 65536) = 16733 nanotokens = 0.000016733 tokens
`


1KB 账户的实际存储费用可能更高，具体取决于合约的具体功能。

如果账户余额低于到期仓储费，则账户被冻结，余额从仓储费中扣除并减为零。 剩余的仓储费作为债务存入账户。

**注意**：当前全局配置可以随时在 [ton.live](http://ton.live/) 的最新关键块细节的主配置部分中查看（[示例](https://net.ton.live/blocks?section=details&id=8cee868a94b1e22794a927279286dc95498310cda982f4657e351a3da693cf27)它只能通过 验证者的投票。

## 消息 Fees
每条消息都需要支付转发费用，该费用根据以下公式计算：
`
msg_fwd_fee = (lump_price + ceil((bit_price * msg.bits + cell_price * msg.cells) / 2 ^ 16))
`

`msg.bits` 和 `msg.cells` 是根据表示为单元树的消息计算的。 不计算根细胞。 `lump_price`、`bit_price`、`cell_price` 包含在全局配置参数 p24 和 p25 中，并且只能并且只能通过验证者的投票来更改。

>**注意**：与存储费用一样，msg.bits 通常很容易估计，而 msg.cells 值可能因不同类型的消息而异。

**示例**：让我们计算在工作链上发送 1KB 消息的最低转发费用：

`lump_price` = 10000000

`bit_price` = 655360000

`cell_price` = 65536000000

为了计算 `msg.bits`，我们从总消息位中减去根单元位。 对于这个例子，我们假设根单元格被完全填满（通常情况并非如此，减去的值更小，这会导致更高的费用）：
`msg.bits` = 8192 - 1023 = 7169

为了计算 msg.cells，我们从单元格总数中减去根单元格。 1 KB 消息中的最小单元数为 9（将 8192/1023 向上舍入为最接近的整数）。 因此 msg.cells 计算如下：

`msg.bits` = 9 - 1 = 8

1KB 消息的最低转发费用计算如下：

`
msg_fwd_fee = (10000000 + ceil((655360000 * 7169 + 65536000000 * 8) / 65536)) = 89690000 nanotokens = 0.08969 tokens
`


1 KB 消息的实际转发费用可能更高，具体取决于消息的类型和内容。
>注意：当前的全局配置总是可以在 [ton.live](http://ton.live/) 的最新关键块详细信息（[示例](https://net.ton.live/blocks?section=details&id=8cee868a94b1e22794a927279286dc95498310cda982f4657e351a3da693cf27))的主配置部分中查看。


### **出站消息**
outbound_internal_messages_fee 计算为交易执行结果生成的每条消息的出站内部消息费用总和：
`
outbound_internal_messages_fee = SUM (out_int_msg.header.fwd_fee + out_int_msg.header.ihr_fee)
`

其中：


out_int_msg.header.fwd_fee 是出站内部消息的标准转发费用的一部分。
out_int_msg.header.ihr_fee 当前已禁用。

#### Routing

出站内部消息的转发费用分为 int_msg_mine_fee 和 int_msg_remain_fee ：

msg_forward_fee = int_msg_mine_fee + int_msg_remain_fee

其中：
`
int_msg_mine_fee = msg_forward_fee * first_frac / 2 ^ 16
`

first_frac - 包含在全局配置参数 p24 和 p25 中，并确定当前验证器集收到的费用的比例。

注意：当前的全局配置总是可以在 [ton.live](http://ton.live/) 的最新关键块详细信息（[示例](https://net.ton.live/blocks?section=details&id=8cee868a94b1e22794a927279286dc95498310cda982f4657e351a3da693cf27))的主配置部分中查看。

int_msg_mine_fee 然后成为交易操作费用的一部分（见下文）。


剩余的 `int_msg_remain_fee` 放置在出站内部消息的标头中（成为 `out_int_msg.header.fwd_fee`），并将转到处理消息的验证器。

如果在转发到目标地址时，消息通过额外的验证器集（即，如果在转发消息时验证器集更改不止一次），则 out_int_msg.header.fwd_fee 的一部分将支付给相关的验证器集 每次和消息头中的剩余费用减少以下金额：

`
intermediate_fee = out_int_msg.header.fwd_fee * next_frac / 2 ^ 16
`

`next_frac` - 包含在全局配置参数 p24 和 p25 中，并确定中间验证器收到的剩余远期费用的比例。


>**注意**：当前的全局配置总是可以在 ton.live 的最新关键块详细信息（示例）的主配置部分中查看。
>**注意**：航线长度不影响远期费的初始计算。 根据全局配置参数，费用在所有涉及的验证器之间简单地分摊。
>**注意**：如果抛出异常并生成退回邮件，则需要收费，就像单个常规出站邮件一样。

### 入站外部消息

每当需要导入入站外部消息以执行交易时，此操作的费用根据[标准转发费用公式计算](https://docs.ton.dev/86757ecb2/v/0/p/632251-fee-calculation-details/t/21b00b)，并支付给当前验证者。

## Action Fees
执行“发送消息”操作需要支付操作费用。 它们包括外部出站消息的所有费用和内部出站消息费用的第一部分。 它们的计算方法如下：
`
total_action_fees = total_out_ext_msg_fwd_fee + total_int_msg_mine_fee
`

其中:

-   `total_out_ext_msg_fwd_fee` - 所有生成的出站外部消息的隐式转发费用的总和。
-   `total_int_msg_mine_fee` - 出站内部消息的“mine”部分消息转发费用的总和。
- `total_fwd_fees` 是一种单独的计算总转发费用的方法。

`
total_fwd_fees = total_action_fees + SUM (int_msg_remain_fee + out_int_msg.header.ihr_fee)
`

`out_int_msg.header.ihr_fee` - 此费用目前为零。

如果在交易期间未执行任何操作，则可能不存在操作费用。

## Gas Fees

`trans.gas_fees`包括与交易相关的所有 gas 费用。 您可以在[gas计算基础](https://docs.ton.dev/86757ecb2/v/0/p/00f1c7-managing-gas-in-ton/t/9171b9) 部分找到更多信息。

与行动费一样，汽油费并不总是存在。 如果 TVM 计算阶段未在事务中初始化，则可以跳过它们。

