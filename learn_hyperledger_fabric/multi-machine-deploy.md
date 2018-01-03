#-------------192.168.180.111单机上部署流程-------------
##1.启动网络
###执行脚本
    cd /root/gopath/src/github.com/hyperledger/fabric-samples/insurance/
	./startFabric.sh
##2.启动express
###先杀掉已有的express进程,再启动
    cd /root/gopath/src/github.com/hyperledger/fabric-samples/insurance
    nohup node server.js &
##3.启动Angular
###在自己机器上启动
    npm start
#------------intel140、intel158多机器部署流程------------
##1.修改/etc/hosts文件，增加下面的代码：
    # For fabric network
    192.168.180.158 peer0.org1.example.com
    192.168.180.158 peer1.org1.example.com
    192.168.180.140 peer0.org2.example.com
    192.168.180.140 peer1.org2.example.com
    192.168.180.158 orderer.example.com
##2.在两台机器上执行清理环境的命令
    cd /root/gopath/src/github.com/hyperledger/fabric-samples/first-network/
    ./byfn.sh -m down
##3.在intel158上运行密码材料等生成脚本
    cd /root/gopath/src/github.com/hyperledger/fabric-samples/first-network/
    ./byfn.sh -m generate
##4.将158上生成的密码材料等拷贝到对应的140目录下
	scp channel-artifacts/* root@intel140:/root/gopath/src/github.com/hyperledger/fabric-samples/first-network/channel-artifacts
    scp -r crypto-config root@intel140:/root/gopath/src/github.com/hyperledger/fabric-samples/first-network
##5.修改158上的docker-compose-cli.yaml
###(1)在 cli 的 volumes 中加入映射关系：
    - /etc/hosts:/etc/hosts
###(2)注释 cli 中的 depends_on 和 command :
    environment:
    #      - CORE_PEER_TLS_ENABLED=true
    #      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
    #      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
    #      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    #command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - orderer.example.com
      - peer0.org1.example.com
      - peer1.org1.example.com
	# - peer0.org2.example.com
	# - peer1.org2.example.com
##6.修改140上的docker-compose-cli.yaml
###(1)在 cli 的 volumes 中加入映射关系：
    - /etc/hosts:/etc/hosts
###(2)注释 cli 中的 depends_on 和 command :
    environment:
    #      - CORE_PEER_TLS_ENABLED=true
    #      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
    #      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
    #      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    #command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
    #  - orderer.example.com
    #  - peer0.org1.example.com
    #  - peer1.org1.example.com
	  - peer0.org2.example.com
	  - peer1.org2.example.com
###(3)修改cli中的环境变量:
    - CORE_PEER_ADDRESS=peer0.org2.example.com:7051
    - CORE_PEER_LOCALMSPID=Org2MSP
    - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
##6.修改140、158两台机器上的yaml：
###(1)base/peer-base.yaml添加 volumes
    volumes:
    - /etc/hosts:/etc/hosts
###这样 peer 容器能通过域名访问orderer了。
###(2)注释掉base/peer-base.yaml中TLS相关的信息
###(3)注释掉docker-compose-cli.yaml中TLS相关的信息
##7.启动网络
###(1)启动158上的docker
    docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d orderer.example.com couchdb0 couchdb1 peer0.org1.example.com peer1.org1.example.com cli
###(2)启动140上的docker
    docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d couchdb2 couchdb3 peer0.org2.example.com peer1.org2.example.com cli
###(3)进入158上cli容器
    docker exec -it cli bash
###(4)创建通道
    peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx
###(5)加入通道
####peer0.org1.example.com加入通道
	peer channel join -b mychannel.block
####peer1.org1.example.com加入通道
	CORE_PEER_ADDRESS=peer1.org1.example.com:8051
    peer channel join -b mychannel.block
####peer0.org2.example.com加入通道
	在158上执行：
	scp /tmp/block/mychannel.block root@intel140:/tmp/block/mychannel.block
	进入cli容器，在140上执行：
	docker exec -it cli bash
    peer channel join -b mychannel.block
####peer0.org2.example.com加入通道
    CORE_PEER_ADDRESS=peer0.org2.example.com:10051
    peer channel join -b mychannel.block
###(6)更新锚点
####org1更新锚点，158执行
    peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx
####org2更新锚点，140执行
    peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx
###(7)安装链码
####首先分别进入cli容器：
    docker exec -it cli bash
####peer0.org1.example.com安装链码
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
	peer chaincode install -n insurance -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/blockchain_insurance
####peer1.org1.example.com安装链码
	CORE_PEER_ADDRESS=peer1.org1.example.com:8051
    peer chaincode install -n insurance -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/blockchain_insurance
####peer0.org2.example.com安装链码
	CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    peer chaincode install -n insurance -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/blockchain_insurance
####peer0.org2.example.com安装链码
    CORE_PEER_ADDRESS=peer0.org2.example.com:10051
    peer chaincode install -n insurance -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/blockchain_insurance
###(8)实例化链码
####任意节点执行
    peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n insurance -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
###(9)初始化链码
    peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n insurance -c '{"Args":["initLedger"]}'
###(10)查询链码
    peer chaincode query -C mychannel -n insurance -c '{"Args":["queryAll","user"]}'
###(11)升级链码版本（实测失败，待后续研究）
####再次安装链码，给予更高的版本
    peer chaincode install -n insurance -v 2.0 -p github.com/hyperledger/fabric/examples/chaincode/go/blockchain_insurance
####升级链码添加变量
    peer chaincode upgrade -o orderer.example.com:7050 -n insurance -v 2.0 -c '{"Args":["initUser","user_20","testUpgrade","test","test@neusoft.com","5","8888"]}'
####查询链码
    peer chaincode query -C mychannel -n insurance -v 2.0 -c '{"Args":["queryAll","user"]}'
##8.启动express
###在140机器执行：
	cd /root/gopath/src/github.com/hyperledger/fabric-samples/insurance/
    ./startExpress.sh
###日志记录在log.txt中。
##9.启动Angular
###修改proxy.conf.json中chaincode指向：
	  "/chaincode":{
	    "target":"http://192.168.180.140:8081"
	  }