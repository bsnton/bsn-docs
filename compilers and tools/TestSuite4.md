######  COMPILERS AND TOOLS
# TestSuite4
TestSuite4 是一个用于简化 TON 合约开发和测试的框架。 它包含轻量级区块链模拟器，可以轻松地以 TDD -Frindly的风格开发合约。  

[Smart Contracts Test Suite 4视频Youtube链接](https://youtu.be/FZMS9_glhXA)

## 特点:

-   **速度** - 在短短几秒钟内执行数十项测试。
-   **复杂的测试场景** - 使用 Python 作为脚本语言允许您创建不同复杂度的测试场景。
-  **深度集成** - 访问所有内部消息，测量gas和控制时间。
-   **易于安装** - 使用 `pip install tonos-ts4` 在本地 Windows、Linux 或 macOS 上安装 TestSuite4。 如果需要，您还可以轻松地从源代码编译它。

有关框架使用的`self-documented`示例，请参阅教程。

## 教程列表

-   [tutorial01_getters.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial01_getters.py) - 使用各种类型的getter。
-   [tutorial02_methods.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial02_methods.py) - 使用外部方法。
-   [tutorial03_constructors.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial03_constructors.py) - 使用构造函数。
-   [tutorial04_messages.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial04_messages.py) - 在合约之间分发消息并捕获事件。
-   [tutorial05_deploy.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial05_deploy.py) - 从合约部署合约并通过“wrappers”使用它。 
-   [tutorial06_signatures.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial06_signatures.py) - 处理外部调用并处理合同引发的异常。
-   [tutorial07_time.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial07_time.py) - “Fast-forwarding”时间，你需要。
-   [tutorial08_balance.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial08_balance.py) - 获取合约余额。
-   [tutorial09_send_money.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial09_send_money.py) - send money并观看它在虚拟区块链中的过程。
-   [tutorial10_encode_call.py](https://github.com/tonlabs/TestSuite4/blob/master/tutorials/tutorial10_encode_call.py) - 对用于 `transfer()` 调用的有效负载进行编码。
