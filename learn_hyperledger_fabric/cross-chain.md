##1.启动网络
###清理环境
    ./byfn.sh -m down
###生成证书、初始区块、创建通道交易等
    ./byfn.sh -m generate
    ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/achannel.tx -channelID achannel
###启动网络
    docker-compose -f dc-orderer-kafka.yaml -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
##2.通过CLI操作fabric网络
###进入cli容器
    docker exec -it cli bash
###分别创建2个通道
    peer channel create -o orderer0:7050 -c mychannel -t 20 -f ./channel-artifacts/channel.tx
    peer channel create -o orderer0:7050 -c achannel -t 20 -f ./channel-artifacts/achannel.tx
###分别加入两个通道，并且查看加入是否成功
    peer channel join -b mychannel.block
    peer channel join -b achannel.block
    peer channel list
###安装链码
    peer chaincode install -n supply_chain -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/mutichannel/supply_chain    
    peer chaincode install -n insurance_main -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/mutichannel/insurance_main
###在不同的通道内分别实例化链码
    peer chaincode instantiate -o orderer0:7050 -C mychannel -n insurance_main -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
    peer chaincode instantiate -o orderer0:7050 -C achannel -n supply_chain -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
###初始化链码
    peer chaincode invoke -o orderer0:7050 -C mychannel -n insurance_main -c '{"Args":["initLedger"]}'
    peer chaincode invoke -o orderer1:7050 -C achannel -n supply_chain -c '{"Args":["initData"]}'
###查询链码
    peer chaincode query -C mychannel -n insurance_main -c '{"Args":["queryById","user_2"]}'
###跨通道查询链码
    peer chaincode query -C achannel -n supply_chain -c '{"Args":["invokeChaincode","insurance_main","mychannel","queryById","user_2"]}'
###跨通道调用链码
    peer chaincode invoke -o orderer1:7050 -C achannel -n supply_chain -c '{"Args":["invokeChaincode","insurance_main","mychannel","initUser","user_98","1","1","1","1","1"]}'
    #####不报错，但是不生效，也就是说跨链只能查询，不能增、删、改#####
###重新启动express
    cd /root/gopath/src/github.com/hyperledger/fabric-samples/insurance
    ./startExpress.sh
##3.前台部署
###登录到140上，进入相关目录
    cd /root/gopath/src/github.com/hyperledger/fabric-samples/blockchain-insurance
###svn更新
	svn update
###打包
	./node_modules/.bin/ng build --prod --aot=false
###拷贝到Apache目录下
	cp -r dist /var/www/html/
###付权
	chmod -R +x /var/www/html/dist
###修改Apache代理
	vi /etc/apache2/sites-enabled/000-default.conf
改成如下所示

    ProxyRequests Off
    ProxyPass /chaincode http://192.168.180.111:8081/chaincode
    ProxyPassReverse /chaincode http://192.168.180.111:8081/chaincode
###重启Apache
	/etc/init.d/apache2 restart
###访问页面
	http://192.168.xxx.xxx/#/auth/login	