# Solidity
Solidity是实现智能合约的高级语言。 它受到C ++，Python和JavaScript的影响，旨在针对以太坊虚拟机（EVM）。

Solidity是静态类型的，支持继承，库和复杂的用户定义类型等功能。

Solidity可以创建投票，众筹，盲拍卖，多重签名钱包等等的合约。
## 智能合约介绍
### 一个简单的智能合约
#### Storage
	pragma solidity ^0.4.0;
	
	contract SimpleStorage {
	    uint storedData;
	
	    function set(uint x) public {
	        storedData = x;
	    }
	
	    function get() public constant returns (uint) {
	        return storedData;
	    }
	}
第一行简单地告诉我们，源代码是为Solidity版本0.4.0编写的，或者是任何不会破坏功能（最高但不包括0.5.0版本）的新版本。 这是为了确保合约不会突然与新的编译器版本有所不同。 关键字pragma是这样调用的，因为一般来说，编译指示是编译器关于如何处理源代码的指令（例如，[pragma once](https://en.wikipedia.org/wiki/Pragma_once)）。

一个Solidity合约是位于以太坊区块链的特定地址的代码（其功能）和数据（其状态）的集合。 `uint storedData;`一行声明了一个类型为`uint`（256位无符号整数）的名为`storedData`的状态变量。 您可以将其视为数据库中的单个插槽，通过调用管理数据库的代码的函数可以查询和更改该插槽。 在以太坊的中，这总是拥有合同。 在这种情况下，函数`set`和`get`可以用来修改或检索变量的值。

要访问一个状态变量，你不需要使用`this.`前缀。 这在其他语言中很常见。

除了允许任何人存储世界上任何人都可以访问的单一号码之外，这个合约还没有做太多的工作可以做（由于以太坊建立的基础设施），没有一个（可行）的方法来阻止你发布这个号码。 当然，任何人都可以再次调用`set`并覆盖你的号码，但号码仍然会保存在区块链的历史记录中。 稍后，我们将看到如何强制进行访问限制，以便只有您可以更改号码。
>**注意：所有标识符（合同名称，函数名称和变量名称）都限制为ASCII字符集。 可以在字符串变量中存储UTF-8编码的数据。**
>
>**警告：使用Unicode文本时要小心，因为类似的（甚至相同的）字符可以具有不同的代码点，因此将被编码为不同的字节数组。**
#### 子货币示例
以下合约将实现加密货币的最简单形式。 可以凭空产生硬币，但是只有创建合同的人才能做到这一点（采取其他实现方式太繁琐了）。 此外，任何人都可以互相发送硬币，而无需使用用户名和密码进行注册 - 所有你需要的是一个以太坊密钥对。

	pragma solidity ^0.4.0;
	
	contract Coin {
	    // 关键字“public”使这些变量可以从外部读取。
	    address public minter;
	    mapping (address => uint) public balances;
	
	    // Events允许轻客户有效地对变化作出反应。
	    event Sent(address from, address to, uint amount);
	
	    //这是构造函数的代码只有在创建合同时才运行。
	    function Coin() public {
	        minter = msg.sender;
	    }
	
	    function mint(address receiver, uint amount) public {
	        if (msg.sender != minter) return;
	        balances[receiver] += amount;
	    }
	
	    function send(address receiver, uint amount) public {
	        if (balances[msg.sender] < amount) return;
	        balances[msg.sender] -= amount;
	        balances[receiver] += amount;
	        Sent(msg.sender, receiver, amount);
	    }
	}
这份合约引入了一些新的概念，让我们一一阐述。

`address public minter;`一行声明一个可公开访问的地址类型的状态变量。 `address`类型是一个不允许任何算术运算的160位值。它适用于存储属于外部人员的合约或密钥对的地址。关键字`public`会自动生成一个函数，允许您访问状态变量的当前值。 没有这个关键字，其他合约就无法访问这个变量。 该函数看起来像这样：

    function minter() returns (address) { return minter; }
当然，添加一个像这样的函数是行不通的，因为我们会有一个名字相同的函数和一个状态变量，但是希望你有这个潜意识 - 编译器会为你设计。

下一行，`mapping (address => uint) public balances;` 也创建一个公共状态变量，但它是一个更复杂的数据类型。 类型将地址映射为无符号整数。 映射可以被看作散列表，它被虚拟初始化，使得每个可能的键都存在并被映射到其字节表示全部为零的值。 但是，这种类比并不太过分，因为既不可能获得映射的所有键的列表，也不能获得所有值的列表。 因此，要么记住（或更好的保留列表或使用更高级的数据类型）添加到映射的内容，要么在不需要的情况下使用它，就像这样。 在这种情况下，`public`关键字创建的`getter`函数有点复杂。 它大致如下所示：
    
	function balances(address _account) public view returns (uint) {
    return balances[_account];
    }
如您所见，您可以使用此功能轻松查询单个帐户的余额。

`event Sent(address from, address to, uint amount);`一行声明了在函数`send`的最后一行中触发的所谓“事件”。 用户界面（当然也包括服务器应用程序）可以在没有太多成本的情况下侦听在区块链上被触发的事件。 一旦被触发，听众也会收到`from`，`to`和`amount`参数，这使得跟踪交易变得容易。 为了听这个事件，你会使用

	Coin.Sent().watch({}, '', function(error, result) {
	    if (!error) {
	        console.log("Coin transfer: " + result.args.amount +
	            " coins were sent from " + result.args.from +
	            " to " + result.args.to + ".");
	        console.log("Balances now:\n" +
	            "Sender: " + Coin.balances.call(result.args.from) +
	            "Receiver: " + Coin.balances.call(result.args.to));
	    }
	})
注意如何从用户界面调用自动生成的`balances`函数。

特别的，`Coin`函数是在创建合约时运行的构造函数，不能在事后调用。它永久存合约创建人的地址：`msg`（还有`tx`和`block`一起）是一个神奇的全局变量，包含一些允许访问区块链的属性。 `msg.sender`始终是当前（外部）函数调用的来源地址。

最后，合约可以由用户和其他合约调用的函数是`mint`和`send`。 如果除创建合约的帐户之外的任何人调用`mint`，那么什么都不会发生。 另一方面，任何人（已经有一些这种硬币的人）都可以调用`send`发送硬币给其他人。 请注意，如果您使用此合约将硬币发送到某个地址，那么当您在区块链浏览器中查看该地址时，您将看不到任何内容，因为您发送硬币和已更改余额的事实仅存储在此特定的硬币合约的数据存储中。通过使用事件，创建追踪新硬币交易和余额的“区块链浏览器”相对容易。
### 区块链基础

