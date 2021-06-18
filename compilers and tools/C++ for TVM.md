###### COMPILERS AND TOOLS

# C++ for TVM
可以使用 Clang-7 支持的任何 C++ 标准编写 C++ 合约。因此，开发人员可以使用最新的 C++17 标准中提供的大部分功能。然而，C++最初并不是为合约而设计的语言，在区块链领域，C++有一些限制：

可以使用 Clang-7 支持的任何 C++ 标准编写 C++ 合同。因此，开发人员可以使用最新的 C++17 标准中提供的大部分功能。然而，由于区块链是领域，C++ 最初并不是为合约而设计的，并且有一些限制：

- 所有算术都是 257-bits wide，sizeof(char) == sizeof() == 1 byte == 257 bits。
- TVM 不支持float(浮点)运算，因此使用 float 或 double 类型的合约编译可能会导致内部编译器错误。
- TVM 在模拟指向函数的指针时性能非常差，因此优化后仍有指针调用的程序将无法编译。因此，我们强烈建议使用具有类和模板的 C 的编码风格，但此存储库的示例中显示了一些例外情况。
- 当前不支持异常处理。

除此之外，我们在语言中引入了一些扩展来处理区块链的细节：
### 扩展语言

-   GNU 属性来标记合约的公共方法（有关详细信息，请参阅此 [repo](https://github.com/tonlabs/samples/tree/master/cpp)中的任何示例）。
-   auto [=a] = 表达式;. 结构绑定扩展允许分配给已经声明的变量 a。 请注意，结构绑定中允许新变量和现有变量的任何混合（例如 auto [=a, b, c, =d] = 表达式；定义 b 和 c 并分配给 a 和 d）。
-   用于生成合约 ABI 的 smart_interface<T> 模板类和宏（有关详细信息，请参阅此存储库[repo](https://github.com/tonlabs/samples/tree/master/cpp) 中的任何示例）
-   纯虚函数声明中的 `void foo() = n` 用于指定合约中的公共方法 id。

请注意，C++ 编译器正在积极开发中，我们的目标是改进诊断，但目前编译可能会导致内部编译器错误。 在这种情况下，请参考上述指南并在社区聊天中寻求帮助。
## C++ 编译工作流程
编译C++智能合约，请执行以下步骤。
1. 提取合约ABI（也可以手工编写，但它复制了易出错的合约接口信息）。 要提取 ABI，请运行
```
clang++ -target tvm --sysroot <path/to/TON-Compiler/stdlib/> Contract.cpp -export-json-abi -o Contract.abi
```
2. 编译合约并link
```
export TVM_LINKER=</path/to/tvm_linker/binary>
export TVM_INCLUDE_PATH=</path/to/TON-Compiler/stdlib>
export TVM_LIBRARY_PATH=</path/to/TON-Compiler/stdlib>
</path/to/TON-Compiler/llvm/tools/tvm-build/tvm-build++.py> --abi Contract.abi Contract.cpp --include $TVM_INCLUDE_PATH
```
如果编译成功，会在 `tvm-build++`运行的目录下创建` *.tvc `文件。
## 部署合约
在这里，我们描述了使用 `tonos-cli` 工具将合约部署到 TON 区块链测试网络（testnet）。 以下命令必须从 `tonos-cli` 所在的目录运行。

0.将 `tonos-cli` 安装目录添加到 PATH
```
export PATH=/path/to/tonos-cli/target/<debug or release>/
```
1.生成合约地址
```
tonos-cli genaddr Contract.tvc Contract.abi --genkey key
```
该命令生成并打印列出合约的所有可能地址：
```
Raw address: <RawAddress>
testnet:
Non-bounceable address (for init): <MyContractNonBounceTestAddress>
Bounceable address (for later access): <MyContractBounceTestAddress>
mainnet:
Non-bounceable address (for init): <MyContractNonBounceMainAddress>
Bounceable address (for later access): <MyContractBounceMainAddress>
```
稍后我们将使用合约的 <RawAddress>。 请注意，--genkey（或--setkey）是强制性的，它会生成一个密钥文件，用于与链下(offchain)应用程序进行的合约的每次交互。 TVM 支持未经授权的外部（即来自区块链外部）方法调用，但 C++ 目前不支持。

2.准备账户。 合约将其代码和数据存储在区块链中，但需要花钱。 所以我们必须在部署合约之前将一些coins转移到未来的合约地址。 **获取测试coins**中介绍了一些获取测试coins的方法。

3.部署合约
```
tonos-cli deploy --abi Contract.abi Contract.tvc '{<constructor call arguments>}' --sign key
```
请注意，调用参数应该是 JSON 格式的“参数”：值对。 例如，'{"a":5, "b":6}'使用 a = 5 和 b = 6 调用构造函数。请注意，参数名称必须与合约 ABI 中指定的参数名称匹配。

4.运行方法 方法的运行在语法上与部署类似，但还应指定方法的名称。
```
tonos-cli call --abi Contract.abi <RawAddress> '{<call arguments>}' --sign key
```
请注意，合约方法也可能在内部调用（即由另一个合约调用），请参阅 Giver 示例了解更多信息。

## 获取测试coins
目前，我们可以描述两种获取测试币的方式：

### 1) 请您的朋友将测试币发送到您的地址。
如果你有一个使用 TON 智能合约的朋友，他们可能会有一些额外的测试币和一个具有转移功能的合约（例如，它可能是一个钱包合约，可以将测试币发送到给定的地址）。 在这种情况下，您可以要求他们向您的地址发送一些测试币。

### 2）使用给予者合同。
如果您知道您的 TON 网络中有一个giver合约，并且您知道它的地址和可能的密钥，您可以要求giver将一些测试币转移到您的地址。

2.1) 如果你需要与你的giver一起使用密钥，请将它们保存到 2 个文件中：
**secret.key** - 64 bytes的密钥和公钥的串联； 
**public.key** - 32 bytes的公钥。

2.2) 使用 tvm_linker 创建消息，我们将调用其函数。 使用giver的 abi 来创建这个消息（在这个例子中giver的 abi 被保存到文件 giver.abi.json 并且函数名和参数是从它中获取的）。  

``` 
tvm_linker message <GiverAddress> -w 0 --abi-json giver.abi.json --abi-method 'sendTransaction' --abi-params '{"dest":"0:<MyContractAddress>","value":"<number of nanograms>","bounce":"false"}' --setkey secret.key
```  

`<GiverAddress>` - 没有工作链 ID 的十六进制格式的giver合约地址。 该命令生成名为 `<*-msg-body.boc>` 的 **.boc** 文件。  

2.3) 使用 lite_client 发送在步骤 **2.2** 中获得的** .boc **文件。 运行 lite_client：
```
lite-client -C ton-lite-client-test1.config.json
```  


发送 **.boc** 文件。
```
sendfile <path_to_file_<*-msg-body.boc>>
```
2.4) 检查余额补充是否成功：我们可以通过不同的方式来进行这个检查：

2.4.1) 使用 ton.live 服务：

使用您的合同地址转到 `https://net.ton.live/accounts?section=details&id=0:<MyContractAddress>`

2.4.2) 使用 GraphQL:

打开 Playground [https://net.ton.live](https://net.ton.live%2C/),并运行以下代码： 
```
{
  accounts
  (filter:{id:{eq:"0:<MyContractAddress>"}})
  {
    acc_type
    balance
    code
  }
}
```

如果成功，您将返回类似的代码：
```
{
  "data": {
    "accounts": [
      {
        "acc_type": 0,
        "balance": "<NumberOfNanograms>",
        "code": null
      }
    ]
  }
}
```
2.4.3) 使用lite_client:

运行 lite_client 并执行以下命令：
```
getaccount 0:<MyContractAddress>
```

如果一切正常，您会得到以下输出：
```
...
           value:(currencies
             grams:(nanograms
               amount:(var_uint len:5 value:<NumberOfNanograms>))
...
```

[原文链接](https://docs.ton.dev/86757ecb2/p/085361-c-for-tvm)
