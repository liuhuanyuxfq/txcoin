# 一、编译智能合约
## 1.编写智能合约，具体代码如下：
    contract Proof
    {
	struct FileDetails{
	    uint timestamp;
	    string owner;
    }
    mapping (string => FileDetails) files;
    event logFileAddedStatus(bool status, uint timestamp, string owner, string fileHash);
    
	//this is used to store the owner of file at the block timestamp
    function set(string owner, string fileHash{
    	//There is no proper way to check if a key already exists or not therefore we are 
	    if(files[fileHash].timestamp == 0){
		    files[fileHash] = FileDetails(block.timestamp, owner);
		    //we are triggering an event so that the frontend of our app knows that the 
		    logFileAddedStatus(true, block.timestamp, owner, fileHash);
	    }
	    else
	    {
		    //this tells to the frontend that file's existence and ownership details couldn't 
		    logFileAddedStatus(false, block.timestamp, owner, fileHash);
	    }
    }
    //this is used to get file information
    function get(string fileHash) returns (uint timestamp, string owner){
    	return (files[fileHash].timestamp, files[fileHash].owner);
    	}
    }
## 2.编译智能合约
使用智能合约的在线编译器https://ethereum.github.io/browser-solidity/编译上面的代码
## 3.修改编译器代码
最终代码如下：

    var proofContract = web3.eth.contract([{"constant":false,"inputs":[{"name":"fileHash","type":"string"}],...","type":"event"}]);
    var proof = proofContract.new(
       {
	     from: web3.eth.accounts[0], 
	     data: '0x6060604052341561000f57...', 
	     gas: '2500000'
       }, function (e, contract){
	    console.log(e, contract);
	    if (typeof contract.address !== 'undefined') {
    	 console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
		}
     })