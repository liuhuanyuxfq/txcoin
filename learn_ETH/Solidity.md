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
### 以太坊虚拟机（EVM）
#### 概述
以太坊虚拟机（EVM）是以太坊智能合约的运行环境。它不仅是沙盒，而且是完全隔离的，这意味着在EVM内运行的代码无法访问网络，文件系统或其他进程。智能合约甚至对其他智能合约的访问也受限。
#### 账户
以太坊有两种共享相同地址空间的帐户：由公私密钥对（即人类）控制的**外部帐户**和由与帐户一起存储的代码控制的**合约帐户**。

外部账户的地址是由公钥确定的，而合同的地址是在创建合同时确定的（它是从创建者地址和从该地址发送的交易数量中导出的，即所谓的“随机数”）。

无论帐户是否存储代码，这两种类型均由EVM平等对待。

每个帐户都有一个持久的键值存储，它将256位字映射为256位字，被称为**Storage**。

此外，每个账户在以太网中都有一个余额（确切的说是“Wei”），是可以通过发送包含以太的交易进行修改的。
#### 交易
交易是从一个帐户发送到另一个帐户（可能是相同的或特殊的零帐户，见下文）的消息。 它可以包含二进制数据（有效负载）和以太。

如果目标账户包含代码，则执行该代码，并将有效负载作为输入数据提供。

如果目标账户是零账户（地址为0的账户），交易将创建一个新的合约。 如前所述，该合约的地址不是零地址，而是源自发件人的地址及其发送的交易数（“随机数”）。 这种合约创建事务的有效负载被认为是EVM字节码并被执行。此执行的输出永久存储为合同的代码。 这意味着为了创建合同，您不会发送合同的实际代码，而是实际返回代码的代码。
#### 天然气（Gas）
在创建之后，每笔交易都会收取一定数量的**Gas**，其目的是限制执行交易所需的工作量并支付执行费用。 当EVM执行交易时，天然气按照特定的规则逐渐耗尽。

**gas price**是由交易的创建者设定的值，其必须从发送`gas_price * gas`的费用来发起交易。如果执行后留下一些gas，则以同样的方式退还。

如果气体被用完（即为负值），则会触发一个气体异常，这将回滚当前调用中对状态所做的所有修改。
#### 存储，内存和堆栈（Storage, Memory and the Stack）
每个帐户都有一个称为**Storage**的永久存储区。存储是将256位字映射到256位字的键值存储。从合约中枚举存储是不可能的，而且读取更加昂贵，更改存储更甚。合同不能读或写任何自身以外的存储。

第二个存储区称为**memory**，合约为每个消息调用获得的一个全新的实例。存储器是线性的，可以在字节级别寻址，但读取限制在256位的宽度，写入可以是8位或256位宽。当访问（读取或写入）先前未触摸​​的内存字（即字中的任何偏移量）时，内存由一个字（256位）扩展。在扩展的同时，必须支付gas的成本。内存越大，成本就越高（它按比例缩放）。

EVM不是注册机，而是堆栈机，所以所有的计算都在一个称为**stack**的区域上执行。它具有1024个元素的最大尺寸并且包含256位的字。堆栈的访问被限制在顶端，可以通过以下方式进行：可以将最顶层的16个元素中的一个复制到堆栈顶部，或者将最顶层的元素与其下面的16个元素中的一个交换。所有其他操作从堆栈中取最上面的两个（或一个或多个，取决于操作）元素，并将结果推送到堆栈上。当然，可以将堆栈元素移动到存储器或内存中，但是不可能仅仅访问堆栈中更深处的任意元素，而不必先移除堆栈的顶部。
#### 指令系统
EVM的指令集保持最小，以避免可能导致共识问题的错误实现。 所有指令都以基本数据类型（256位字）进行操作。 通常的算术，比特，逻辑和比较操作都存在。 有条件的和无条件的跳转是可能的。 此外，合约可以像访问其编号和时间戳一样访问当前块的相关属性。
#### 消息调用
合同可以通过消息调用的方式调用其他合同或发送以太到非合同账户。消息调用类似于事务，因为它们有一个源，一个目标，数据有效负载，以太，天然气和返回数据。实际上，每个事务都由一个顶级消息调用组成，这个消息调用又可以创建更多的消息调用。

合同可以决定剩余的天然气有多少发送给内部的信息和有多少要保留。如果在内部调用（或任何其他异常）中发生了一个`out-of-gas`异常，这将通过放置在堆栈上的错误值来指示。在这种情况下，与调用一起发送的气体被用完。在Solidity中，在这种情况下，发起调用的合约默认会导致手动异常，所以异常会“冒泡”到调用堆栈。

如前所述，被调用合约（可以与发起方相同）将收到一个全新的内存实例，并可以调用有效负载 - 将在一个称为**calldata**的单独区域中提供。在完成执行后，它可以返回将被存储在由调用者预先分配的调用者的存储器中的位置处的数据。

调用被限制在1024层的深度，这意味着对于更复杂的操作，循环应该优于递归调用。
#### Delegatecall / Callcode and Libraries
消息调用存在一个特殊的变体，名为**delegatecall**，它与消息调用相同，除了目标地址上的代码在调用合约的上下文中执行，并且`msg.sender`和`msg.value`不会更改他们的价值。

这意味着合同可以在运行时动态加载来自不同地址的代码。 存储，当前地址和余额仍然是指向发起合同，只有代码来自被叫地址。

这使得在Solidity中实现“库”功能成为可能：可重用的库代码可以应用于合同的存储，例如， 以实现一个复杂的数据结构。
#### 日志
可以将数据存储在专门索引的数据结构中，该数据结构一直映射到块级别。 这个被称为**日志**的特性被Solidity用来实现 **events**。 合同创建后无法访问日志数据，但是可以从区块链之外高效访问它们。 由于日志数据的某些部分存储在[bloom过滤器](https://en.wikipedia.org/wiki/Bloom_filter)中，因此可以以高效且密码安全的方式搜索这些数据，因此不下载整个区块链（“轻客户端”）的网络对等方仍然可以找到这些日志。
#### 建立
合同甚至可以使用特殊的操作码来创建其他合同（即它们不是简单地调用零地址）。这些创建调用和正常消息调用之间唯一的区别是有效负载数据被执行，结果作为代码存储，调用者/创建者在堆栈上接收新契约的地址。
#### 自毁
从区块链中删除代码的唯一可能性是该地址的合同执行`selfdestruct`操作。存储在该地址的剩余Ether被发送到指定的目标，然后存储和代码从状态中移除。

**警告：即使合同的代码不包含对`selfdestruct`的调用，它仍然可以使用`delegatecall`或`callcode`执行该操作。**

注意：旧合同的修剪可能会也可能不会被以太坊客端户实现。 另外，归档节点可以选择无限期地保持合同存储和代码。
注意：目前外部帐户不能从状态中删除。
## 安装Solidity
### Remix
如果你只是想尝试使用Solidity编写简单合同，你可以尝试[Remix](https://remix.ethereum.org/)，这种方式不需要安装。 如果您想在不连接互联网的情况下使用它，您可以访问[https://github.com/ethereum/browser-solidity/tree/gh-pages]()并按照该页面上的说明下载.ZIP文件。
### npm / Node.js
这可能是本地安装Solidity最便携和最方便的方法。

通过使用Emscripten将C++源代码编译成JavaScript，提供了一个与平台无关的JavaScript库。 它可以直接用于项目（如Remix）。 请参阅[solc-js](https://github.com/ethereum/solc-js)存储库以获取相关说明。

还有一个名为*solcjs*的命令行工具，它可以通过npm来安装：

    npm install -g solc
注意：solcjs的comandline选项与solc和工具（如geth）不兼容，希望solc的行为不会与solcjs一起工作。
### Docker
我们为编译器提供了最新的docker版本。 `stable`版本库包含已发布的版本，而`nightly`版本库包含开发分支中潜在不稳定变化的版本。

	docker run ethereum/solc:stable solc --version
目前，docker镜像只包含编译器可执行文件，所以你必须做一些额外的工作来链接源码和输出目录。
### 二进制包
可用的Solidity二进制包在[solidity/releases](https://github.com/ethereum/solidity/releases)。

我们也有Ubuntu的PPA。使用最新的稳定版本：

	sudo add-apt-repository ppa:ethereum/ethereum
	sudo apt-get update
	sudo apt-get install solc
如果你想使用开发分支的版本：

	sudo add-apt-repository ppa:ethereum/ethereum
	sudo add-apt-repository ppa:ethereum/ethereum-dev
	sudo apt-get update
	sudo apt-get install solc
我们还发布了一个snap包，它可以安装在所有受支持的Linux发行版中。 要安装最新的solc版本：

	sudo snap install solc
或者，如果你想帮助测试不稳定的solc与来自开发分支的最新变化：
	
	sudo snap install solc --edge
Arch Linux也有包，尽管只限于最新的开发版本：
	
	pacman -S solidity
### 从源代码构建
## Solidity实例
### 投票
以下合同相当复杂，但展示了很多Solidity的特点。 它执行投票合约。 当然，电子投票的主要问题是如何给正确的人分配投票权，以及如何防止操纵。 我们不会在这里解决所有的问题，但至少我们会展示如何进行委派投票，以便计票是自动的，同时是完全透明的。

这个想法是为每个选票创建一个合约，为每个选项提供一个简短的名字。 然后，担任主席的合约创造者将分别给予每个地址投票权。

地址背后的人可以选择自己投票，也可以把他们的投票委托给他们信任的人。

在投票结束时，`winningProposal（）`将返回投票数最多的提案。

	pragma solidity ^0.4.16;
	
	/// @title Voting with delegation.
	contract Ballot {
	    // This declares a new complex type which will
	    // be used for variables later.
	    // It will represent a single voter.
	    struct Voter {
	        uint weight; // weight is accumulated by delegation
	        bool voted;  // if true, that person already voted
	        address delegate; // person delegated to
	        uint vote;   // index of the voted proposal
	    }
	
	    // This is a type for a single proposal.
	    struct Proposal {
	        bytes32 name;   // short name (up to 32 bytes)
	        uint voteCount; // number of accumulated votes
	    }
	
	    address public chairperson;
	
	    // This declares a state variable that
	    // stores a `Voter` struct for each possible address.
	    mapping(address => Voter) public voters;
	
	    // A dynamically-sized array of `Proposal` structs.
	    Proposal[] public proposals;
	
	    /// Create a new ballot to choose one of `proposalNames`.
	    function Ballot(bytes32[] proposalNames) public {
	        chairperson = msg.sender;
	        voters[chairperson].weight = 1;
	
	        // For each of the provided proposal names,
	        // create a new proposal object and add it
	        // to the end of the array.
	        for (uint i = 0; i < proposalNames.length; i++) {
	            // `Proposal({...})` creates a temporary
	            // Proposal object and `proposals.push(...)`
	            // appends it to the end of `proposals`.
	            proposals.push(Proposal({
	                name: proposalNames[i],
	                voteCount: 0
	            }));
	        }
	    }
	
	    // Give `voter` the right to vote on this ballot.
	    // May only be called by `chairperson`.
	    function giveRightToVote(address voter) public {
	        // If the argument of `require` evaluates to `false`,
	        // it terminates and reverts all changes to
	        // the state and to Ether balances. It is often
	        // a good idea to use this if functions are
	        // called incorrectly. But watch out, this
	        // will currently also consume all provided gas
	        // (this is planned to change in the future).
	        require((msg.sender == chairperson) && !voters[voter].voted && (voters[voter].weight == 0));
	        voters[voter].weight = 1;
	    }
	
	    /// Delegate your vote to the voter `to`.
	    function delegate(address to) public {
	        // assigns reference
	        Voter storage sender = voters[msg.sender];
	        require(!sender.voted);
	
	        // Self-delegation is not allowed.
	        require(to != msg.sender);
	
	        // Forward the delegation as long as
	        // `to` also delegated.
	        // In general, such loops are very dangerous,
	        // because if they run too long, they might
	        // need more gas than is available in a block.
	        // In this case, the delegation will not be executed,
	        // but in other situations, such loops might
	        // cause a contract to get "stuck" completely.
	        while (voters[to].delegate != address(0)) {
	            to = voters[to].delegate;
	
	            // We found a loop in the delegation, not allowed.
	            require(to != msg.sender);
	        }
	
	        // Since `sender` is a reference, this
	        // modifies `voters[msg.sender].voted`
	        sender.voted = true;
	        sender.delegate = to;
	        Voter storage delegate = voters[to];
	        if (delegate.voted) {
	            // If the delegate already voted,
	            // directly add to the number of votes
	            proposals[delegate.vote].voteCount += sender.weight;
	        } else {
	            // If the delegate did not vote yet,
	            // add to her weight.
	            delegate.weight += sender.weight;
	        }
	    }
	
	    /// Give your vote (including votes delegated to you)
	    /// to proposal `proposals[proposal].name`.
	    function vote(uint proposal) public {
	        Voter storage sender = voters[msg.sender];
	        require(!sender.voted);
	        sender.voted = true;
	        sender.vote = proposal;
	
	        // If `proposal` is out of the range of the array,
	        // this will throw automatically and revert all
	        // changes.
	        proposals[proposal].voteCount += sender.weight;
	    }
	
	    /// @dev Computes the winning proposal taking all
	    /// previous votes into account.
	    function winningProposal() public view
	            returns (uint winningProposal)
	    {
	        uint winningVoteCount = 0;
	        for (uint p = 0; p < proposals.length; p++) {
	            if (proposals[p].voteCount > winningVoteCount) {
	                winningVoteCount = proposals[p].voteCount;
	                winningProposal = p;
	            }
	        }
	    }
	
	    // Calls winningProposal() function to get the index
	    // of the winner contained in the proposals array and then
	    // returns the name of the winner
	    function winnerName() public view
	            returns (bytes32 winnerName)
	    {
	        winnerName = proposals[winningProposal()].name;
	    }
	}
### 盲拍卖
在本节中，我们将展示在以太坊上创建完全匿名的拍卖合同是多么容易。 我们将从一个公开的拍卖开始，每个人都可以看到所做的投标，然后将这个合同扩展到一个盲拍卖，在竞标期结束之前不可能看到实际的出价。
#### 简单的公开拍卖
以下简单拍卖合同的总体思路是每个人都可以在投标期内出价。 出价已经包括发送钱/以太为了约束投标人的投标。 如果最高出价提高了，以前最高的出价者就拿回了她的钱。 在投标期结束后，合同必须手动叫收款人收取他的钱 - 合同不能激活自己。

	pragma solidity ^0.4.11;
	
	contract SimpleAuction {
	    // Parameters of the auction. Times are either
	    // absolute unix timestamps (seconds since 1970-01-01)
	    // or time periods in seconds.
	    address public beneficiary;
	    uint public auctionEnd;
	
	    // Current state of the auction.
	    address public highestBidder;
	    uint public highestBid;
	
	    // Allowed withdrawals of previous bids
	    mapping(address => uint) pendingReturns;
	
	    // Set to true at the end, disallows any change
	    bool ended;
	
	    // Events that will be fired on changes.
	    event HighestBidIncreased(address bidder, uint amount);
	    event AuctionEnded(address winner, uint amount);
	
	    // The following is a so-called natspec comment,
	    // recognizable by the three slashes.
	    // It will be shown when the user is asked to
	    // confirm a transaction.
	
	    /// Create a simple auction with `_biddingTime`
	    /// seconds bidding time on behalf of the
	    /// beneficiary address `_beneficiary`.
	    function SimpleAuction(
	        uint _biddingTime,
	        address _beneficiary
	    ) public {
	        beneficiary = _beneficiary;
	        auctionEnd = now + _biddingTime;
	    }
	
	    /// Bid on the auction with the value sent
	    /// together with this transaction.
	    /// The value will only be refunded if the
	    /// auction is not won.
	    function bid() public payable {
	        // No arguments are necessary, all
	        // information is already part of
	        // the transaction. The keyword payable
	        // is required for the function to
	        // be able to receive Ether.
	
	        // Revert the call if the bidding
	        // period is over.
	        require(now <= auctionEnd);
	
	        // If the bid is not higher, send the
	        // money back.
	        require(msg.value > highestBid);
	
	        if (highestBidder != 0) {
	            // Sending back the money by simply using
	            // highestBidder.send(highestBid) is a security risk
	            // because it could execute an untrusted contract.
	            // It is always safer to let the recipients
	            // withdraw their money themselves.
	            pendingReturns[highestBidder] += highestBid;
	        }
	        highestBidder = msg.sender;
	        highestBid = msg.value;
	        HighestBidIncreased(msg.sender, msg.value);
	    }
	
	    /// Withdraw a bid that was overbid.
	    function withdraw() public returns (bool) {
	        uint amount = pendingReturns[msg.sender];
	        if (amount > 0) {
	            // It is important to set this to zero because the recipient
	            // can call this function again as part of the receiving call
	            // before `send` returns.
	            pendingReturns[msg.sender] = 0;
	
	            if (!msg.sender.send(amount)) {
	                // No need to call throw here, just reset the amount owing
	                pendingReturns[msg.sender] = amount;
	                return false;
	            }
	        }
	        return true;
	    }
	
	    /// End the auction and send the highest bid
	    /// to the beneficiary.
	    function auctionEnd() public {
	        // It is a good guideline to structure functions that interact
	        // with other contracts (i.e. they call functions or send Ether)
	        // into three phases:
	        // 1. checking conditions
	        // 2. performing actions (potentially changing conditions)
	        // 3. interacting with other contracts
	        // If these phases are mixed up, the other contract could call
	        // back into the current contract and modify the state or cause
	        // effects (ether payout) to be performed multiple times.
	        // If functions called internally include interaction with external
	        // contracts, they also have to be considered interaction with
	        // external contracts.
	
	        // 1. Conditions
	        require(now >= auctionEnd); // auction did not yet end
	        require(!ended); // this function has already been called
	
	        // 2. Effects
	        ended = true;
	        AuctionEnded(highestBidder, highestBid);
	
	        // 3. Interaction
	        beneficiary.transfer(highestBid);
	    }
	}
#### 盲拍卖
以前的公开拍卖会延伸到下面的一个盲目拍卖。盲目拍卖的好处是，在招标期结束时没有时间压力。在一个透明的计算平台上创建一个盲目的拍卖可能听起来像是一个矛盾，但是密码学就可以拯救。

在投标期间，投标人实际上并没有发出投标书，而只是一个散列版本。由于目前认为实际上不可能找到两个（足够长的）哈希值相等的值，所以投标人通过该投标来投标。在投标期结束后，投标人必须公开投标：未加密地发送其价值，并且合同检查散列值与投标期间提供的散列值相同。

另一个挑战是如何使拍卖会同时具有约束力和盲目性：唯一的方法是防止投标人在获得拍卖后不发送货币，那就是让投标人把它与投标一起发送。既然价值转移不能在以太坊蒙蔽，任何人都可以看到价值。

以下合同通过接受任何大于最高出价的价值来解决此问题。因为这当然只能在披露阶段检查，所以有些出价可能是无效的，这是有意的（它甚至提供了一个明确的标志，以无效的投标高价值转让）：投标人可以混淆竞争，低无效出价。

	pragma solidity ^0.4.11;
	
	contract BlindAuction {
	    struct Bid {
	        bytes32 blindedBid;
	        uint deposit;
	    }
	
	    address public beneficiary;
	    uint public biddingEnd;
	    uint public revealEnd;
	    bool public ended;
	
	    mapping(address => Bid[]) public bids;
	
	    address public highestBidder;
	    uint public highestBid;
	
	    // Allowed withdrawals of previous bids
	    mapping(address => uint) pendingReturns;
	
	    event AuctionEnded(address winner, uint highestBid);
	
	    /// Modifiers are a convenient way to validate inputs to
	    /// functions. `onlyBefore` is applied to `bid` below:
	    /// The new function body is the modifier's body where
	    /// `_` is replaced by the old function body.
	    modifier onlyBefore(uint _time) { require(now < _time); _; }
	    modifier onlyAfter(uint _time) { require(now > _time); _; }
	
	    function BlindAuction(
	        uint _biddingTime,
	        uint _revealTime,
	        address _beneficiary
	    ) public {
	        beneficiary = _beneficiary;
	        biddingEnd = now + _biddingTime;
	        revealEnd = biddingEnd + _revealTime;
	    }
	
	    /// Place a blinded bid with `_blindedBid` = keccak256(value,
	    /// fake, secret).
	    /// The sent ether is only refunded if the bid is correctly
	    /// revealed in the revealing phase. The bid is valid if the
	    /// ether sent together with the bid is at least "value" and
	    /// "fake" is not true. Setting "fake" to true and sending
	    /// not the exact amount are ways to hide the real bid but
	    /// still make the required deposit. The same address can
	    /// place multiple bids.
	    function bid(bytes32 _blindedBid)
	        public
	        payable
	        onlyBefore(biddingEnd)
	    {
	        bids[msg.sender].push(Bid({
	            blindedBid: _blindedBid,
	            deposit: msg.value
	        }));
	    }
	
	    /// Reveal your blinded bids. You will get a refund for all
	    /// correctly blinded invalid bids and for all bids except for
	    /// the totally highest.
	    function reveal(
	        uint[] _values,
	        bool[] _fake,
	        bytes32[] _secret
	    )
	        public
	        onlyAfter(biddingEnd)
	        onlyBefore(revealEnd)
	    {
	        uint length = bids[msg.sender].length;
	        require(_values.length == length);
	        require(_fake.length == length);
	        require(_secret.length == length);
	
	        uint refund;
	        for (uint i = 0; i < length; i++) {
	            var bid = bids[msg.sender][i];
	            var (value, fake, secret) =
	                    (_values[i], _fake[i], _secret[i]);
	            if (bid.blindedBid != keccak256(value, fake, secret)) {
	                // Bid was not actually revealed.
	                // Do not refund deposit.
	                continue;
	            }
	            refund += bid.deposit;
	            if (!fake && bid.deposit >= value) {
	                if (placeBid(msg.sender, value))
	                    refund -= value;
	            }
	            // Make it impossible for the sender to re-claim
	            // the same deposit.
	            bid.blindedBid = bytes32(0);
	        }
	        msg.sender.transfer(refund);
	    }
	
	    // This is an "internal" function which means that it
	    // can only be called from the contract itself (or from
	    // derived contracts).
	    function placeBid(address bidder, uint value) internal
	            returns (bool success)
	    {
	        if (value <= highestBid) {
	            return false;
	        }
	        if (highestBidder != 0) {
	            // Refund the previously highest bidder.
	            pendingReturns[highestBidder] += highestBid;
	        }
	        highestBid = value;
	        highestBidder = bidder;
	        return true;
	    }
	
	    /// Withdraw a bid that was overbid.
	    function withdraw() public {
	        uint amount = pendingReturns[msg.sender];
	        if (amount > 0) {
	            // It is important to set this to zero because the recipient
	            // can call this function again as part of the receiving call
	            // before `transfer` returns (see the remark above about
	            // conditions -> effects -> interaction).
	            pendingReturns[msg.sender] = 0;
	
	            msg.sender.transfer(amount);
	        }
	    }
	
	    /// End the auction and send the highest bid
	    /// to the beneficiary.
	    function auctionEnd()
	        public
	        onlyAfter(revealEnd)
	    {
	        require(!ended);
	        AuctionEnded(highestBidder, highestBid);
	        ended = true;
	        beneficiary.transfer(highestBid);
	    }
	}
### 安全的远程购买

	pragma solidity ^0.4.11;
	
	contract Purchase {
	    uint public value;
	    address public seller;
	    address public buyer;
	    enum State { Created, Locked, Inactive }
	    State public state;
	
	    // Ensure that `msg.value` is an even number.
	    // Division will truncate if it is an odd number.
	    // Check via multiplication that it wasn't an odd number.
	    function Purchase() public payable {
	        seller = msg.sender;
	        value = msg.value / 2;
	        require((2 * value) == msg.value);
	    }
	
	    modifier condition(bool _condition) {
	        require(_condition);
	        _;
	    }
	
	    modifier onlyBuyer() {
	        require(msg.sender == buyer);
	        _;
	    }
	
	    modifier onlySeller() {
	        require(msg.sender == seller);
	        _;
	    }
	
	    modifier inState(State _state) {
	        require(state == _state);
	        _;
	    }
	
	    event Aborted();
	    event PurchaseConfirmed();
	    event ItemReceived();
	
	    /// Abort the purchase and reclaim the ether.
	    /// Can only be called by the seller before
	    /// the contract is locked.
	    function abort()
	        public
	        onlySeller
	        inState(State.Created)
	    {
	        Aborted();
	        state = State.Inactive;
	        seller.transfer(this.balance);
	    }
	
	    /// Confirm the purchase as buyer.
	    /// Transaction has to include `2 * value` ether.
	    /// The ether will be locked until confirmReceived
	    /// is called.
	    function confirmPurchase()
	        public
	        inState(State.Created)
	        condition(msg.value == (2 * value))
	        payable
	    {
	        PurchaseConfirmed();
	        buyer = msg.sender;
	        state = State.Locked;
	    }
	
	    /// Confirm that you (the buyer) received the item.
	    /// This will release the locked ether.
	    function confirmReceived()
	        public
	        onlyBuyer
	        inState(State.Locked)
	    {
	        ItemReceived();
	        // It is important to change the state first because
	        // otherwise, the contracts called using `send` below
	        // can call in again here.
	        state = State.Inactive;
	
	        // NOTE: This actually allows both the buyer and the seller to
	        // block the refund - the withdraw pattern should be used.
	
	        buyer.transfer(value);
	        seller.transfer(this.balance);
	    }
	}