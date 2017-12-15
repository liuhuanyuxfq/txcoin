# 一、环境构建
## 1.连接到交互式环境
	geth attach /root/eth/data/p0/geth.ipc
## 2.启动挖矿
	miner.start()
# 二、编译智能合约
## 1.编写智能合约，具体代码如下：
	contract HelloWorld
	{
	    address creator;
	    string greeting;
	
	    function HelloWorld (string _greeting) payable
	    {
	        creator = msg.sender;
	        greeting = _greeting;
	    }
	
	    function greet() constant returns (string)
	    {
	        return greeting;
	    }
	
	    function setGreeting(string _newgreeting)
	    {
	        greeting = _newgreeting;
	    }
	
	     /**********
	     Standard kill() function to recover funds
	     **********/
	
	    function kill()
	    {
	        if (msg.sender == creator)
	            suicide(creator);  // kills this contract and sends remaining funds back to creator
	    }
	}
## 2.编译智能合约
使用智能合约的在线编译器https://ethereum.github.io/browser-solidity/编译上面的代码
![](2.png)
### 注意：第一行的`pragma solidity ^0.4.0;`必须保留，用于指定版本
## 3.修改编译器代码
### 点击网页上的Details按钮，在web3Deploy框里点选复制代码，得到代码如下：

    var _greeting = /* var of type string here */ ;
    var browser_ballot_sol_helloworldContract = web3.eth.contract([{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_newgreeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"}]);
    var browser_ballot_sol_helloworld = browser_ballot_sol_helloworldContract.new(
       _greeting,
       {
     from: web3.eth.accounts[0], 
     data: '0x60606040526040516104c53803806104c583398101604052808051820191905050336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff160217905550806001908051906020019061007692919061007d565b5050610122565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100be57805160ff19168380011785556100ec565b828001600101855582156100ec579182015b828111156100eb5782518255916020019190600101906100d0565b5b5090506100f991906100fd565b5090565b61011f91905b8082111561011b576000816000905550600101610103565b5090565b90565b610394806101316000396000f300606060405260043610610057576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806341c0e1b51461005c578063a413686214610071578063cfae3217146100ce575b600080fd5b341561006757600080fd5b61006f61015c565b005b341561007c57600080fd5b6100cc600480803590602001908201803590602001908080601f016020809104026020016040519081016040528093929190818152602001838380828437820191505050505050919050506101ed565b005b34156100d957600080fd5b6100e1610207565b6040518080602001828103825283818151815260200191508051906020019080838360005b83811015610121578082015181840152602081019050610106565b50505050905090810190601f16801561014e5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614156101eb576000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b565b80600190805190602001906102039291906102af565b5050565b61020f61032f565b60018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156102a55780601f1061027a576101008083540402835291602001916102a5565b820191906000526020600020905b81548152906001019060200180831161028857829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106102f057805160ff191683800117855561031e565b8280016001018555821561031e579182015b8281111561031d578251825591602001919060010190610302565b5b50905061032b9190610343565b5090565b602060405190810160405280600081525090565b61036591905b80821115610361576000816000905550600101610349565b5090565b905600a165627a7a7230582022a13c27a817a3ef7bc7491955f10f52f0b3535b713d872d385a5e23141f23790029', 
     gas: '4700000'
       }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
     console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
     })
### 修改第一行：
    var _greeting = "Hello World" ;   //你想打招呼的内容
### 修改第八行，gas设置：
	gas: '470000'
# 三、部署和执行代码
## 1.解锁账户
	personal.unlockAccount(eth.accounts[0],"1q2w3e")
## 2.在交互式环境中将修改后的代码粘贴进去,并等待出块
![](3.png)
## 3.运行合约
	browser_ballot_sol_helloworld.greet()
![](4.png)
# 四、参考资料
solidity编译：[https://solidity.readthedocs.io/en/develop/using-the-compiler.html]()

交互式环境管理API：[https://github.com/ethereum/go-ethereum/wiki/Management-APIs]()

JavaScript API：[https://github.com/ethereum/wiki/wiki/JavaScript-API]()

browser-Solidity文档：[https://github.com/ethereum/browser-solidity/]()