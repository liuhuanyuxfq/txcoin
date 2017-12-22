[准备工作](../prepare/contents.md)
#一、下载工具和docker镜像
##1.新建目录：
	mkdir fabric-sample
	cd fabric-sample
##2.执行shell脚本，下载工具和镜像
vi NIKLiU.sh

    #!/bin/bash
	export ARCH=$(echo "$(uname -s|tr '[:upper:]' '[:lower:]'|sed 's/mingw64_nt.*/windows/')-$(uname -m | sed 's/x86_64/amd64/g')" | awk '{print tolower($0)}')

	curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/fabric-binary/${ARCH}-1.0.0-alpha2/fabric-binary-${ARCH}-1.0.0-alpha2.tar.gz | tar xz
	cd release/${ARCH}
	sh download-dockerimages.sh -c $(uname -m)-1.0.0-alpha2 -f $(uname -m)-1.0.0-alpha2
#二、密码生成器
Cryptogen使用包含网络拓扑的文件`crypto-config.yaml`，并允许我们为组织和属于这些组织的组件生成一个证书库。 每个组织都配置了唯一的根证书（`ca-cert`），它将特定组件（peers and orderers）绑定到该组织。 fabric内的交易和通信由实体的私钥（`keystore`）签名，然后通过公钥（`signcerts`）进行验证。 您将在该文件中注意到一个“count”变量。 我们使用它来指定每个组织的peers数量; 在我们这个例子中，它是每个组织的两个对等体。 这个模板的其余部分是非常自明的。  
在我们运行该工具之后，这些证书将被停放在一个名为`crypto-config`的文件夹中。
#三、配置事务生成器
`configtxgen tool`用于创建四个配置工件：orderer引导块，结构通道配置事务和两个锚点对等事务 - 每个对等组件一个。

    configtx.yaml  cryptogen.yaml  genesis.block  mychannel.tx
`genesis.block`是order服务的创世区块，`mychannel.tx`会广播。

`Configtxgen`会生成一个包含网络定义的文件`configtx.yaml`。有三个成员 - 一个Orderer组织（OrdererOrg）和两个对等组织（Org1＆Org2），每个Peer组织管理和维护两个对等节点。该文件还指定了一个由我们的两个对等组织组成的联盟-`SampleConsortium`。请特别注意此文件顶部的“Profiles”部分。 你会注意到我们有两个唯一的标题。 一个为创世区块-`TwoOrgsOrdererGenesis`-一个为渠道-`TwoOrgsChannel`。 这些标题很重要，因为我们将在创建我们的artifacts时将它们作为参数传递给它们。 此文件还包含两个值得注意的附加规定。 首先，我们为每个对等体组织（peer0.org1.example.com＆peer0.org2.example.com）指定锚点对等体。 其次，我们指向每个成员的MSP目录的位置，用来在orderer创世区块存储每个组织的根证书。这是一个关键的概念。现在与orderer服务通信的任何网络实体可以验证其数字签名。  
为了方便使用，提供了一个脚本 - `generateArtifacts.sh`。 该脚本将生成加密材料和我们的四个配置工件，并随后将这些文件输出到`channel-artifacts`文件夹中。
#四、运行密码生成工具和事务生成工具
##1.手动生成
### 进入相应的目录：  
	cd /fabric-sample/release/linux-amd64
###运行密码生成命令：
    ./bin/cryptogen generate --config=./crypto-config.yaml
###会生成如下警告，是正常的：  

    2017-06-08 16:25:20.681 CST [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
###接下来指定`configtxgen`到哪里去找运行需要的`configtx.yaml`文件：
    export FABRIC_CFG_PATH=$PWD
###创建orderer创世区块：
    ./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
###创建交易渠道工件：
    # make sure to set the <channel-ID> parm
    
    ./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mytestchl
###在渠道上为Org1定义一个锚点：
    ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mytestchl -asOrg Org1MSP
###在渠道上为Org2定义一个锚点：
    ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mytestchl -asOrg Org2MSP
##2.脚本生成：
###先删除之前生成的文件：
    ./network_setup.sh down
###执行脚本生成文件（脚本包含手动生成的所有步骤）：
    ./generateArtifacts.sh <channel-ID>
#五、启动网络
###注释掉`docker-compose-cli.yaml`中的script.sh脚本行，以达到手动执行并研究的目的
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    # command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
    volumes
###启动网络：
    CHANNEL_NAME=mytestchl TIMEOUT=100000 docker-compose -f docker-compose-cli.yaml up -d
###环境变量：
目前的cli连接的是`peer0.org1.example.com`，如果想连接其他peer，修改docker-compose-cli.yaml中相应的环境变量  

    # Environment variables for PEER0
    
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
#六、创建和加入渠道
###进入cli容器
`docker exec -it cli bash`	
###创建渠道
**注意：`-c`指定渠道名，`-f`指定交易渠道工件，就是之前生成的`./channel-artifacts/channel.tx`**

    peer channel create -o orderer.example.com:7050 -c mytestchl -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem
上述命令生成了渠道的创世区块-`<channel-ID.block>`-我们以后要通过它加入这个渠道
###加入渠道
    peer channel join -b mytestchl.block
#七、将链码安装和实例化
##安装
    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
##初始化链码（在这步中指定背书策略）
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem -C mytestchl -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
##查询
    peer chaincode query -C mytestchl -n mycc -c '{"Args":["query","a"]}'
##调用
    peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem  -C mytestchl -n mycc -c '{"Args":["invoke","a","b","10"]}'
#八、脚本进行调用
###清空环境
    ./network_setup.sh down
###执行全自动脚本
    ./network_setup.sh up <channel-ID> <timeout-value>
上述脚本将依次执行`generateArtifacts.sh`、`script.sh`。
其中，`script.sh`脚本依次创建渠道、加入渠道、更新锚点、安装链码、初始化链码、查询链码、调用链码。脚本非常适合做shell学习的范本。
###实验证实了什么
链码必须安装在对等体上，以便成功地对分类帐执行读/写操作。 此外，直到针对该链码执行init或传统事务 - 读/写 - （例如，查询“a”的值）时，对等体才会启动链码容器。 事务导致容器启动。 此外，通道中的所有对等体都保留分类帐的精确副本，其中包含以块形式存储不可变顺序的记录，以及状态数据库以维持当前的结构状态。 这包括那些没有在其上安装链码的对等体（如上例中的peer1.org1.example.com）。 最后，链式代码在安装后可以访问（如上述示例中的peer1.org2.example.com），因为它已被实例化。
###查看链码日志的方法：
    root@ubuntu:/fabric-sample/release/linux-amd64# docker logs dev-peer0.org2.example.com-mycc-1.0
    ex02 Init
    Aval = 100, Bval = 200
    root@ubuntu:/fabric-sample/release/linux-amd64# docker logs dev-peer1.org2.example.com-mycc-1.0
    ex02 Invoke
    Query Response:{"Name":"a","Amount":"90"}
    root@ubuntu:/fabric-sample/release/linux-amd64# docker logs dev-peer0.org1.example.com-mycc-1.0
    ex02 Invoke
    Query Response:{"Name":"a","Amount":"100"}
    ex02 Invoke
    Aval = 90, Bval = 210
###使用couchdb跑marble例子
    #清理环境
    ./byfn.sh -m down
    #生成证书等
    ./byfn.sh -m generate
    #启动网络
    CHANNEL_NAME=mychannel TIMEOUT=10000 docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
    #进入cli
    docker exec -it cli bash
    #设定环境变量
    export CHANNEL_NAME=mychannel
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    #创建channel，加入channel，设定锚点
    ./scripts/pre.sh
    
    #部署链码
    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
    peer chaincode install -n mycc1 -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
    
    peer chaincode install -n marblesor -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02

    peer chaincode install -n insurance -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/blockchain_insurance
    #初始化链码
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR	('Org1MSP.member','Org2MSP.member')"
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc1 -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND	('Org1MSP.member','Org2MSP.member')"
    
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"
    #调用链码
    #创建3个marble
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -c '{"Args":["initMarble","marble2","red","50","tom"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
    #移动marble
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -c '{"Args":["transferMarble","marble2","jerry"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
    #删除marble
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marblesor -c '{"Args":["delete","marble1"]}'
    #查询
    peer chaincode query -C $CHANNEL_NAME -n marblesor -c '{"Args":["readMarble","marble1"]}'
    peer chaincode query -C $CHANNEL_NAME -n marblesor -c '{"Args":["getMarblesByRange","marble1","marble4"]}'
    peer chaincode query -C $CHANNEL_NAME -n marblesor -c '{"Args":["queryMarblesByOwner","tom"]}'
    #查询历史
    peer chaincode query -C $CHANNEL_NAME -n marblesor -c '{"Args":["getHistoryForMarble","marble1"]}'
###如果想要持久化存储，那么需要在docker-compose文件中增加下面两条：
####在peer0.org1.example.com中增加
    volumes:
      - /var/hyperledger/peer0org1:/var/hyperledger/production
####在couchdb中增加
    volumes:
      - /var/hyperledger/couchdb:/opt/couchdb/data

#九、node sdk使用
###Download Marbles
    git clone https://github.com/IBM-Blockchain/marbles.git --depth 1
    git checkout v3.0
###Download Docker images
    cd marbles/scripts
    chmod +x download-dockerimages.sh
    sudo ./download-dockerimages.sh
###Set up the Fabric Node SDK
    sudo bash setup_sdk.sh
###Start your network
    cd ./fabric-sdk-node/test/fixtures
    sudo docker-compose -f docker-compose-marblesv3.yaml up -d
###remove any key value stores and SDK artifacts
    rm -rf /tmp/hfc-*
    rm -rf ~/.hfc-key-store
###Create channel
    cd ../fabric-sdk-node/
    node test/integration/e2e/create-channel.js
###Join channel
    node test/integration/e2e/join-channel.js
###安装依赖
	cd <marbles-root>
    npm install
####如果安装期间中断过，可能会安装不完整，需要删除node_modules文件夹并执行
    npm cache clean
####以便清除缓存
###Install chaincode
    cd ./scripts
    node install_chaincode.js
###Instantiate chaincode
    node instantiate_chaincode.js
###Run Marbles
    gulp marbles3
###在本地浏览器打开网址进行操作
    http://192.168.180.157:3001
#十couch-db的使用
如果您选择将fabric-couchdb容器端口映射到主机端口，请确保您知道安全性影响。 在开发环境中映射端口使CouchDB REST API可用，并允许通过CouchDB Web界面（Fauxton）对数据库进行可视化。 生产环境可能不会实施端口映射，以限制对CouchDB容器的外部访问。







---
#代理设置
###git代理设置
    git config --global http.proxy http://username:passwd@proxy_ip:port
###git协议端口受限，改成https协议的方法：
    git config --global url.https://github.com/.insteadOf git://github.com/
###Docker代理设置
*建立配置目录*  

`mkdir /etc/systemd/system/docker.service.d`  
*创建代理配置文件*  

	`vi  /etc/systemd/system/docker.service.d/http-proxy.conf`  

    [Service]
    Environment="HTTP_PROXY=http://username:passwd@proxy_ip:port"
*刷新*  

    systemctl daemon-reload
*查看环境变量以确认配置生效*  

    systemctl show --property=Environment docker
*重启Docker*   

    systemctl restart docker
#Docker国内镜像设置
修改`/etc/docker/daemon.json`（没有时新建该文件）  

    {
    "registry-mirrors": ["https://0433qmq2.mirror.aliyuncs.com"]
    }
#npm 代理设置：  
    npm config set proxy http://username:passwd@proxy_ip:port 
    npm config set https_proxy http://username:passwd@proxy_ip:port
#npm国内镜像设置
    npm config set registry https://registry.npm.taobao.org
设置好后通过`npm config get registry`查看当前的registry