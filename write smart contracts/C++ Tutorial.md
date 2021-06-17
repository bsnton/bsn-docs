WRITE SMART CONTRACTS
# C++ 教程
>**注意**: 本教程正在进行中。 它会在 TVM 的 C++ 获得多汁的新功能后更新，因此我们建议不时返回它。

## 先决条件

要重现本教程，您需要获取 [TON-Compiler sources](https://github.com/tonlabs/TON-Compiler), [TONOS-CLI](https://github.com/tonlabs/tonos-cli), [TVM-linker sources](https://github.com/tonlabs/TVM-linker) 并构建 `clang++`、`tvm_linker` 和 `tonos-cli`。 请遵循相应存储库中的“README”文件。

## 工具链说明
C++ 工具链由以下库和工具组成：
-   C++ 运行时位于`TON-Compiler-source/stdlib/stdlib_cpp.tvm`。
-  `TON-Compiler-source/stdlib/cpp-sdk/std` 中的 C++ std 标头。这些大多是来自 LLVM libstdc++ 的原始头文件，配置了 TVM 架构参数。您可能希望这些标头能够工作并使用它们。但我们不能保证每一个特性都能顺利运行：一些 C++ 特性超出了 TVM 架构的能力。我们稍后将在本教程中讨论这个主题。
-  可用于合约的 Boost hana 库。 它位于 `TON-Compiler-source/stdlib/cpp-sdk/boost`。
-   TON SDK 基于头文件的库，包含与合约一起使用的基本函数和类。 它位于 `TON-Compiler-source/stdlib/cpp-sdk/tvm`。
-   基于 Clang-7 的 C++ 编译器。 二进制文件位于 `TON-Compiler-build/bin/clang++`.
-   `tvm_linker` 用于 TVM 组装的 tvm_linker 链接器，也是用于合约的本地测试工具。 该工具位于 `TVM-linker-source/tvm_linker/target/<debug or release>/tvm_linker`.
-   `tonos-cli` 合约部署工具。 该工具位于 `tonos-cli/target/<debug or release>/tonos-cli`.
-  C++ 的编译器驱动程序。 由于 TVM 后端对优化管道有非常具体的要求，我们强烈建议使用该工具而不是手动调用 clang。 驱动程序位于 `TON-Compiler-build/bin/tvm-build++`.  

## TON和TVM及术语简介

与常见的 C++ 程序不同，TON智能合约是用在区块链中运行。 这意味着代码和数据可能需要永久存储，除非有人修改或销毁它们。 在 TON 区块链中，所有合约彼此之间交换消息。 交换本质上是异步的。 当合约收到消息时，它会解析它，如果消息对合约敏感，它通常会调用一个用户定义的处理程序，称为公共方法（通常是具有公共访问说明符的类方法，但它没有 以这种方式实现）。

当本教程提到公共方法时，它指的是 TVM 方面的公共方法，而不是 C++ 方面的。 一个合约可能会向另一个合约发送消息，并通过此消息异步调用公共方法。 这样的消息称为内部消息（即发送方在区块链中）。

或者，外部工具也可能创建并向合约发送消息。 这种消息称为外部消息。 当合约处理消息时，它的计算资源有限，以 gas 衡量。 Gas 价格从合约余额中扣除，该余额与部署到网络中的合约相关联。

无论何种方式，即使合约有未清余额，它也有一个硬性限制，它可以在单个传入消息上花费多少，如果超过此限制，即使它运行的代码格式良好，合约也会终止。 为了最小化在合约之间传输并存储在区块链中的状态，TVM 没有随机存取存储器。 内存可能会被字典模拟，但是，考虑到有限的气体供应，它非常昂贵且通常不实用。

因此，典型的 C++ 智能合约避免使用内存。 此外，目前所有功能都必须内联。 这就是为什么 -O3 优化和 LTO 是必不可少的，它是 tvm-build++ 实现的管道。

## 环境设置

在开始开发合约之前，我们配置 PATH 以添加我们需要的所有工具：
`
export PATH=TON-Compiler-build/bin:TVM-linker-source/tvm_linker/target/<debug or release>:tonos-cli/target/<debug or release>:$PATH
`

tvm-build++ 需要额外处理：
`
export TVM_LINKER=TVM-linker-source/tvm_linker/target/<debug or release>/tvm_linker #path to tvm_linker tool.
export TVM_INCLUDE_PATH=TON-Compiler-source/stdlib #path to the folder containing cpp-sdk directory.
export TVM_LIBRARY_PATH=TON-Compiler-source/stdlib #path to stdlib_cpp.tvm
`

## Hello, world!
典型的智能合约由两个文件组成：
-   头文件中的合约接口描述
-   cpp -flie中的合约实现 如果需要，合约可能包含更多文件，但由于合约往往很小，因此两个文件通常就足够了。 让我们开始开发我们的第一个，通过`Hello，world！`合约来描述接口。

### 描述合约接口
一个合约接口由三部分组成：
-   公共生命方法.
-   持久数据.
-   事件. 公共方法是在区块链中接收消息的函数。 有一个称为` constructor`的专用方法，它在合约部署时调用。 构造函数是必须的，否则合约无法部署。 构造函数以及其他公共方法只能采用下面列出的类型的参数：

    -   `int_t<N>` - N-bits 宽的有符号整数, 0 < N < 256.
    -   `uint_t<N>` - N-bits 位宽的无符号整数, 0 < N < 257.
    -   `MsgAddress` – 适用于内部或外部消息的消息地址.
    -   `MsgAddressInt` – 内部消息地址。
    -   `MsgAddressExt` – 外部消息地址。
    -   `dict_array<T, 32>` - 表示存储在字典中的“数组”的映射，具有 32 -bits宽的键，这些keys是数组的索引。 `T `也必须属于这个列表。
    -   `dict_map<KeyT, ValueT>` - 一张地图。 `ValueT` 也必须属于该列表，`KeyT` 必须是 `uint_t<N>`。
    -   `sequence<uint_t<8>>` - 一个 8 位无符号值序列，它也可能被视为一个 8 位值的“数组”； 使用顺序存储的数据然后映射通常更便宜。
    -   `lazy<T>` - `T` 的值，但解析被推迟到需要实际值时。 目前只支持Only lazy, lazy和lazy。
    -   复合类型 – 一个 POD 结构，它只有列表中类型的数据成员。 当使用复合类型时，它的编码方式与其元素的序列相同。 另请注意，为了 ABI 生成（见下文），使用了成员的名称，因此请注意可能的冲突。 除此之外，公共方法可能具有任意签名。 但是，目前用于 TVM 的 C++ 不支持公共方法模板。 对于 Hello, world contract 我们只需要构造函数和一个发送外部消息的方法：
```
// Hello world interface
struct IHelloWorld {
  // Handle external messages only
  __attribute__((external))
  void constructor() = 1;

  // Handle external messages only
  __attribute__((external))
  uint_t<8> hello_world() = 2;
};
```
这里所有的方法都是纯虚方法，`= N`中的N表示它们的ID。 这些 ID 在contract中必须是唯一的，并且 ID 也不能等于 0。此外，方法必须至少标记为以下属性之一：
- external – 该方法处理外部传入消息
- internal – 该方法处理内部传入的消息
- getter - 该方法可以在链下执行，因此您可以在不付费的情况下读取持久数据。 您可能已经注意到，公共方法可能会返回。 在这种情况下，返回值被解释为需要通过外部消息发送的值，因此链下应用程序可以处理它。 返回值的类型也必须属于上面的列表。 请注意，TVM 支持多个返回值。 结构返回类型是使用 C++ 特性的方式。 你好，世界！ 合约不需要任何持久数据成员，但是，目前，它需要至少有一个数据字段，否则您会收到一条看起来很奇怪的错误消息。 hello world！contract ，我们使用以下持久数据：
```
// Hello world persistent data
struct DHelloWorld {
  uint_t<1> x;
};
```
最后，事件可能被解释为一种链下函数调用。 当一个公共方法发出一个事件时，它的工作方式类似于另一个公共方法调用，暗示包含函数 ID 和调用参数的消息被发出。 但是，在发生事件的情况下，此消息将被发送到外部（即在区块链之外），因此外部句柄可以对其进行处理。 事件也是纯虚方法，但不应定义或调用它们。 此外，它们的 ID 必须与公共方法的 ID 不同。 对于 Hello, world contract 我们根本不需要事件：
```
// Hello world events
struct EHelloWorld {};
```
综上所述，以下是 Hello, world contract 接口的完整列表。
```
#pragma once

#include <tvm/schema/message.hpp>

namespace tvm { namespace schema {

// Hello world interface
struct IHelloWorld {
  // Handle external messages only
  __attribute__((external))
  void constructor() = 1;

  // Handle external messages only
  __attribute__((external))
  uint_t<8> hello_world() = 2;
};

// Hello world persistent data
struct DHelloWorld {
  uint_t<1> x;
};

// Hello world events
struct EHelloWorld {};

}} // namespace tvm::schema
```

### 实现
对实现合约，一些SDK 标头会有所帮助。

-   tvm/schema/message.hpp 定义在 TON 中使用的消息结构。
-   tvm/contract.hpp 实现合约类和必要的辅助功能来使用它。
-   tvm/smart_switcher.hpp 实现 smart_interface，它生成用于解析传入消息、序列化方法结果等的样板代码。
-   tvm/replay_attack_protection/timestamp.hpp 实现基于时间戳的重放保护，这是防止同一消息被多次处理的必要条件。 在包含标头之后，我们声明一个代表合约的类。
```
using namespace tvm::schema;
using namespace tvm;

class HelloWorld final : public smart_interface<IHelloWorld>,
                         public DHelloWorld {
…
};
```

为了利用智能切换器不自己编写消息解析，合约类必须从 `start_interface<T>` 继承，其中 `T`是描述公共方法（即接口）的结构体的类型和描述合约持久数据的结构体。 `HelloWorld` 的实现很简单：
```
public:
  __always_inline void constructor() final {}
  __always_inline uint_t<8> hello_world() final {return uint_t<8>(42);};

  // Function is called in case of unparsed or unsupported func_id
static __always_inline int _fallback(cell msg, slice msg_body) { return 0; };
```
**注意:**
1.  `__always_inline` 在这里是必不可少的。 请记住，如果函数内联失败，编译器将失败。
2.  我们已经看到的前两种方法。 除了做所写的，他们总是执行重放保护检查并接受传入的消息。 通过接受消息，联系人同意为其处理付费，因此合约进行的计算不会被丢弃。
3.  当消息 ID 无效或不存在时（例如，当发送它的合约不使用 ABI 时）调用最后一个方法。 智能切换器无法帮助此方法解析消息，也不会在其中插入接受。 因此，通过在那里什么都不做，合约会忽略-格式错误的传入外部消息。 定义合约类后需要添加一些内容：
```
DEFINE_JSON_ABI(IHelloWorld, DHelloWorld, EHelloWorld);
```
插入生成合约所需的 ABI 文件所需的逻辑。
```
DEFAULT_MAIN_ENTRY_FUNCTIONS(HelloWorld, IHelloWorld, DHelloWorld, 1800)
```
生成将控制流转移到公共方法的入口点函数。 这里的 1800 是配置重放保护的参数。 1800 是 ABI 手册推荐的以秒为单位的时间。 综上所述，这里是合约实现的完整清单。
```
#include "HelloWorld.hpp"

#include <tvm/contract.hpp>
#include <tvm/smart_switcher.hpp>
#include <tvm/replay_attack_protection/timestamp.hpp>

using namespace tvm::schema;
using namespace tvm;

class HelloWorld final : public smart_interface<IHelloWorld>,
                         public DHelloWorld {
public:
  __always_inline void constructor() final {}
  __always_inline uint_t<8> hello_world() final {return uint_t<8>(42);};

  // Function is called in case of unparsed or unsupported func_id
  static __always_inline int _fallback(cell msg, slice msg_body) { return 0; }; };
DEFINE_JSON_ABI(IHelloWorld, DHelloWorld, EHelloWorld);

// ----------------------------- Main entry functions ---------------------- //
DEFAULT_MAIN_ENTRY_FUNCTIONS(HelloWorld, IHelloWorld, DHelloWorld, 1800)
```
### 编译和本地测试
到目前为止，我们编写的代码可以在 [Hello, world!](https://github.com/tonlabs/samples/blob/master/cpp/HelloWorld)中找到。假设我们将合约接口放入“HelloWorld.hpp”，并将其实现放入“HelloWorld.cpp”。 编译包括两个步骤：
1.  生成 ABI
```
clang++ -target tvm --sysroot=$TVM_INCLUDE_PATH -export-json-abi -o HelloWorld.abi HelloWorld.cpp
```
1.  编译和链接
```
tvm-build++.py --abi HelloWorld.abi HelloWorld.cpp --include $TVM_INCLUDE_PATH --linkerflags="--genkey key"
```
第一个命令生成 ABI 文件。 请注意，如果合约使用不受支持的类型，clang 将在 ABI 中默默地为其生成“unknow”，并且合约将不会链接。 第二个命令编译并链接合约。 它生成 address.tvc 文件和名为“key”的文件。Option `--genkey` 生成keys以签署消息并将它们存储到指定的文件（`<filename>` - 用于私钥，`<filename.pub>` 用于公共密钥）。 目前所有外部消息都必须签名，否则会出现 40 号错误，因此如果我们打算在本地测试合约，我们需要一个密钥来签名。
### 本地调试
tvm_linker 工具可用于向合约发送消息和从合约获取消息。 但请注意，合约会修改其存储在 tvc 文件中的持久数据，因此此类合约可能无法再部署到网络中。 因此，如果您计划在本地测试合约，我们建议您制作一份合约副本，或者在部署之前重新编译。 在链接器中调试时，构造函数不会自动调用，它应该由程序员显式调用。 如果构造函数没有像 `Hello, world!`不是必须的，但为了演示起见，我们仍然称之为。
```
$ export ADDRESS=`ls *.tvc | cut -f 1 -d '.'`
$ tvm_linker test $ADDRESS \
--abi-json HelloWorld.abi \
--abi-method constructor \
--abi-params '{}' \
--sign key
```
执行合约后，链接器打印一条长消息。 其中最重要的部分是 TVM 以 0 退出代码终止，即成功。 第二个重要部分是消耗了多少gas。 让我们调用`hello_world`方法以确保它也能正常工作。
```
tvm_linker test $ADDRESS \
--abi-json HelloWorld.abi \
--abi-method hello_world \
--abi-params '{}'  \
--sign key \
--decode-c6
```
这里我们使用 `--decode-c6` 选项（请参阅 `tvm_tools –help` 以获取有关其命令行选项的完整手册）来显示传出消息并确保合约确实发送包含 42 作为有效负荷消息。 您将在链接器输出的末尾找到传出消息。 消息体以十六进制形式显示，0x2a = 42。

### 在网中部署和测试
在网络中测试有点类似于本地测试，但需要使用 `tonos-cli` 而不是链接器，并且参数传递有点不同。 部署工作流程在 [README](https://github.com/tonlabs/samples/tree/master/cpp#contract-deployment) 但我们将在这里再次重复。 首先，我们需要重新编译合约，因为我们用于链接器测试。 然后将新生成的tvc文件（为了简单起见重命名为`HelloWorld.tvc`）和abi文件复制到`tonos-cli/target/<debug or release>/` 准备好之后，我们就可以执行下面的脚本了。
```
cd tonos-cli/target/<debug or release>/
cargo run genaddr HelloWorld.tvc HelloWorld.abi --genkey hw.key
```
后一个命令返回合约的原始地址。 现在您可以使用中描述的任何方法向它发送（测试）coins[README](https://github.com/tonlabs/samples/tree/master/cpp#getting-test-coins). 当合约余额大于 0 时，我们可以部署合约：
```
cargo run deploy --abi HelloWorld.abi HelloWorld.tvc '{}' --sign hw.key
```
最后测试 `hello_world`方法：
```
cargo run call –abi HelloWorld.abi "<raw address>" hello_world "{}" --sign hw.key
```
该命令应该输出以
```
Succeded.
Result = {"output":{"value0":"0x2a"}}
```
## 授权

### 接口的实现
现在我们准备扩展我们刚刚开发的合约。 `hello，world!`的主要问题！ 合约是 `hello_world` 公共方法可以被任何人调用。 因为方法执行不是免费的，陌生人可能会向合约发送垃圾邮件，从而花掉所有的余额。 为了防止这种情况，我们需要在接受传入的外部消息之前引入额外的检查。 要执行此检查，合约将执行以下操作:
1. 部署时存储所有者的公钥。
2. 调用 hello_world 时，根据密钥检查消息签名。 实现很简单。 首先，我们需要向合约添加持久数据。
```
// Hello world persistent data
struct DHelloWorld {
  uint_t<256> ownerKey;
};
```
其次，我们需要在 `Hello World`合约类中添加以下成员：
```
// The compiler reads the key from the incoming message and stores is in
// pubkey_. So the key is available in a public method via tvm_pubkey().
unsigned pubkey_ = 0;
__always_inline void set_tvm_pubkey(unsigned pubkey) { pubkey_ = pubkey; }
__always_inline unsigned tvm_pubkey() const { return pubkey_; }
```
第三，我们需要修改构造函数和`hello_world`方法：
```
/// Deploy the contract.
__always_inline void constructor() final { ownerKey = tvm_pubkey(); }
__always_inline uint_t<8> hello_world() final {
  require(tvm_pubkey() == ownerKey, 101);
  return uint_t<8>(42);
}
```
这里我们将公钥存储在构造函数中，然后在`hello_world`中进行检查。 require调用中的`101`是错误代码，需要大于`100`以免干扰虚拟机和C++ SDK错误代码。 唯一的问题是合约在检查需求之前仍然接受传入的消息。 因此，如果陌生人调用它，我们不会为传出消息付费，但仍需为其余的执行付费。 为了解决这个问题，我们需要更改纯虚方法声明：
```
__attribute__((external, noaccept))
uint_t<8> hello_world() = 2;
```
我们还需要明确接受：
```
__always_inline uint_t<8> hello_world() final {
  require(tvm_pubkey() == ownerKey, 101);
  tvm_accept();
  return uint_t<8>(42);
};
```
### 在网络中部署和测试
这次我们省略了链接器中的测试（您可以自己完成，按照 `Hello, world!`合同相应小节中的说明进行操作）。 为了在网络中进行测试，我们在部署时生成另一个密钥，然后检查是否可以使用此密钥和前一个无效的密钥获得结果。 重新编译并复制合约到 `tonos-cli/target/<debug or release>/` 类似于之前的合约。 然后运行：
```
cd tonos-cli/target/<debug or release>/
cargo run genaddr HelloWorld.tvc HelloWorld.abi --genkey auth.key
#send coins to the contract address somehow
cargo run deploy --abi HelloWorld.abi HelloWorld.tvc '{}' --sign auth.key
cargo run call –abi HelloWorld.abi "<raw address>" hello_world "{}" --sign hw.key
cargo run call –abi HelloWorld.abi "<raw address>" hello_world "{}" --sign auth.key
```
第一次调用将失败并超时终止。 在接受之前发生的任何未捕获的异常都不会显示，因为当前节点不支持这样的功能。 要正确诊断它，您应该安装 TON OS SE 并将其用于调试，这超出了本教程的范围。 第二次调用应该成功返回 0x2a。
## 信息交换

### 接口和实现
最后，我们准备实施更复杂的合约，以在彼此之间交换消息。 Giver 是此类合同的一个很好的例子。 我们要求读者熟悉代码。 这里我们只提供一些要点：
1. 内部公共方法可以像外部方法一样声明，但它用 `internal` 属性标记。 内部方法使用传入的消息余额而不是合约的余额来计算，因此与外部方法不同，它不需要接受消息。
2. `tvm/contract_handle.hpp` 提供调用另一个合约方法的函数。 为此，它需要被调用方合约的接口类定义，这就是合约实现的 cpp 和 hpp 部分分开的原因。 语法一般如下：
```
auto handle = contract_handle<ICallee>(callee_address);
handle(message_balance, message_flags).call<&ICallee::method_name>(parameters…);
```
第一行构造合约的句柄。 尽管它可能会调用合同。 第二行通过 `operator()` 配置调用，然后执行它。 `operator()` 是可选的，默认情况下，此配置保证如果发送方有足够的余额，则消息将携带 1 000 000 个单位的货币。
### 本地调试
与之前的测试场景不同，我们需要检查内部消息的工作方式。 为此，首先我们需要生成一条传出消息。 让我们调用 `Client` 合约的 `get_money` 方法，向地址 `<Giver address>` 的 `Giver` 询问 42 000 个货币单位：
```
tvm_linker test <Client address> \
--abi-json Client.abi \
--abi-method get_money \
--abi-params "{\"giver\":\"<Giver address>\", \"balance\":42000}" \
--sign client \
--decode-c6
```
执行后，消息被编码在输出的最后一行之一中：
```
body  : bits: 288   refs: 0   data: 00000002000000000000000000000000000000000000000000000000000000000000a410
```
消息是 0000000200000000000000000000000000000000000000000000000000000000000a410 链接器不会自动发送这条消息给Giver合约，所以我们需要：
```
tvm_linker test <Giver address> \
--src "<Client address> " \
--internal <message balance> \
--body 00000002000000000000000000000000000000000000000000000000000000000000a410 \
--decode-c6
```
`--src` 这里是发件人地址。 如果接收者不检查消息的来源，则可能会省略它。

### 在网络中部署和测试

在真实网络中进行测试时，您不需要发送内部消息——只需要发送外部消息。 所以过程差别不大。 但是，[ton.live](https://net.ton.live/) 对于查看合约的所有传入和传出消息变得必不可少。 您只需要指定原始地址，然后查看日志。

 

 - [ ] [英文链接](https://docs.ton.dev/86757ecb2/p/828241-c-tutorial)
