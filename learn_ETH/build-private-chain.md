本次安装的主机环境是Ubuntu16.0，具体安装步骤如下：
# 一、安装
## 1.安装
    sudo apt-get install software-properties-common
    sudo add-apt-repository -y ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install ethereum
## 2.安装结果测试
    geth version
出现命令行使用帮助信息，说明安装成功
# 二、配置创世区块
    mkdir eth
    cd eth
    vi genesis.json
genesis.json内容如下：

    {
      "config": {
		    "chainId": 314590,
		    "homesteadBlock": 0,
		    "eip155Block": 0,
		    "eip158Block": 0
   		 },
      "alloc"  : {},
      "coinbase"   : "0x0000000000000000000000000000000000000000",
      "difficulty" : "0x80000",
      "extraData"  : "",
      "gasLimit"   : "0x2fefd8",
      "nonce"  : "0x0000000000000042",
      "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
      "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
      "timestamp"  : "0x00"
    }
参数解释：

| 参数名称| 参数描述|
| ------ |:------:|
| alloc| 用来预置账号以及账号的以太币数量|
| coinbase| 矿工的账号，随便填|
| extraData| 附加信息，随便填|
| gasLimit| 该值设置对GAS的消耗总量限制，用来限制区块能包含的交易信息总和。|
| nonce| nonce就是一个64位随机数，用于挖矿，注意他和mixhash的设置需要满足以太坊的Yellow paper, 4.3.4. Block Header Validity, (44)章节所描述的条件。|
| mixhash| 与nonce配合用于挖矿，由上一个区块的一部分生成的hash。注意他和nonce的设置需要满足以太坊的Yellow paper, 4.3.4. Block Header Validity, (44)章节所描述的条件。|
| parentHash| 上一个区块的hash值，因为是创世块，所以这个值是0|
| parentHash| 设置创世块的时间戳|
 
# 三、单节点启动
## 1.初始化
    geth init genesis.json
可以在初始化时指定data目录：

    geth --datadir ./data/p0 --networkid 314590 --port 61910 --rpcport 8200 init genesis.json
## 2.启动
	geth --datadir ./data/p0 --networkid 314590 --port 61910 --rpcport 8200
## 3.在另一个窗口下启动交互式环境
    geth attach /root/eth/data/p0/geth.ipc
## 4.在交互式环境下进行如下操作
#### 创建账户
	personal.newAccount("1q2w3e")   #指定密码，每次只能创建一个账户
#### 启动挖矿
	miner.start()
#### 停止挖矿
	miner.stop()
#### 查看余额
	eth.getBalance(eth.accounts[0])
	eth.getBalance(eth.accounts[1])
#### 解锁账户
	personal.unlockAccount(eth.accounts[0],"1q2w3e")
#### 交易ether
	eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(3,"ether")})
	eth.pendingTransactions
#### 启动挖矿,确认交易
	miner.start(1)
	miner.stop()
	eth.getBalance(eth.accounts[0])
	eth.getBalance(eth.accounts[1])	
	eth.pendingTransactions
#### 主要命令
    eth.accounts      #查看账户
	personal.newAccount("1q2w3e")    #创建账户
	eth.blockNumber    #查看区块数
	miner.start()	#开始挖矿
	miner.stop()	#停止挖矿
	eth.getBalance(eth.accounts[0])		#查看账户余额
	eth.getBalance("0x8b8e9d6616cfa80ec521d976a3498fb12654a5b2")    #查看账户余额
    personal.unlockAccount("0x8b8e9d6616cfa80ec521d976a3498fb12654a5b2","1q2w3e")	#解锁
    eth.sendTransaction({from:"0x8b8e9d6616cfa80ec521d976a3498fb12654a5b2",to:"0x587e57a516730381958f86703b1f8e970ff445d9",value:web3.toWei(3,"ether")})	#转账
	admin.nodeInfo	#查看当前节点
	net.peerCount	#查看peer数量
	admin.peers	#查看peer详情
	web3.toWei(5,'ether') 	#将ether转换成wei，1ether=10^18wei
	web3.fromWei(eth.getBalance(eth.accounts[0]),'ether')  #将wei转换成ether
	txpool.status	#交易池状态
	miner.start(1);admin.sleepBlocks(1);miner.stop(); #启动挖矿，然后等待挖到一个区块之后就停止挖矿
	eth.getTransaction("0x0c59f431068937cbe9e230483bc79f59bd7146edc8ff5ec37fea6710adcab825")  #通过交易hash查看交易
	eth.getBlock(33) 	#通过区块号查看区块
	admin.nodeInfo.enode    #获取节点实例的enode url

![](3.png)

# 四、集群启动
## 1.清理环境：
	rm -rf /root/eth/data
## 2.启动第一个节点
	geth --datadir ./data/p0 --networkid 314590 --port 61910 --rpcport 8200  init genesis.json
    geth --datadir ./data/p0 --networkid 314590 --port 61910 --rpcport 8200
	admin.nodeInfo.enode  #获取节点实例的enode url
## 3.启动第二个节点
	geth --datadir ./data/p1 init genesis.json
	geth --datadir ./data/p1 --networkid 314590 --port 61911 --rpcport 8201 --bootnodes "enode://454de4856f01253dc70fb6120be9f491183027e7b3a6ed7f44524bb2923fe27842475f6647556883330e147c79788266c09daa2ff6f3bbcddc02b66d1fccfaaf@192.168.180.140:61910"
上面的命令中,--bootndoes 是设置当前节点启动后,直接通过设置--bootndoes 的值来链接第一个节点, --bootnoedes 的值可以通过在第一个节的命令行中,输入:admin.nodeInfo.enode命令打印出来.
也可以不设置 --bootnodes, 直接启动,启动后进入命令行, 通过命令admin.addPeer(enodeUrlOfFirst Instance)把它作为一个peer添加进来.
## 4.分别在两个节点创建账户：
	personal.newAccount("1q2w3e")
## 5.peer0启动挖矿：
	miner.start(1)
	miner.stop()
	eth.getBalance(eth.accounts[0])
## 6.交易ether
	personal.unlockAccount(eth.accounts[0],"1q2w3e")
	eth.sendTransaction({from: eth.accounts[0], to: "0x1acf15a3850e4c6b37e71cf15b769ea30604482f", value: web3.toWei(3,"ether")})
	eth.pendingTransactions
## 7.启动挖矿,确认交易
	miner.start(1)
	miner.stop()
	eth.getBalance(eth.accounts[0])
	eth.pendingTransactions
# 五、常识

1. 以太坊成功挖到一个块，奖励10ether
2. Gas是交易计算步骤的计量单位。每笔交易中都包含gas上限和愿意支付的每个gas的费用。矿工如果选择包含这笔交易，那么会获得执行交易消耗的gas对应的费用。如果交易消耗的gas超过gas上限，那么交易内容回滚，但是交易费用（最大gas和gas价格）会付给矿工。每个矿工会设置gas的单价，如果交易中给出的单价低于矿工的单价设置，那么矿工不会执行交易。gas单价以wei作为单位。EVM会对不同的操作定好单价。智能合约所消耗的费用有调用者承担。
3. 以太坊单位换算：
#
- 1 Ether = 1000000000000000000 Wei
- 1 Ether = 1000000000000000 Kwei
- 1 Ether = 1000000000000 Mwei
- 1 Ether = 1000000000 Gwei
- 1 Ether = 1000000 Szabo
- 1 Ether = 1000 Finney
- 1 Ether = 0.001 Kether
- 1 Ether = 0.000001 Mether
- 1 Ether = 0.000000001 Gether
- 1 Ether = 0.000000000001 Tether



