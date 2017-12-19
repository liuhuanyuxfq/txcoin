# Hyperledger fabric的链码测试
## 1.安装Hyperledger Fabric Samples
1. 安装[Hyperledger Fabric Samples](http://hyperledger-fabric.readthedocs.io/en/latest/samples.html)
2. 进入克隆的`fabric-samples`目录下的`chaincode-docker-devmode`目录
## 2.确保下载Docker镜像
上述安装步骤会完成docker镜像的下载，可以用下面的命令查看镜像是否都下载完成：

	root@ub02:~/gopath/src/github.com/hyperledger/fabric-samples/chaincode-docker-devmode# docker images
	REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
	hyperledger/fabric-tools       latest              0403fd1c72c7        5 months ago        1.32GB
	hyperledger/fabric-tools       x86_64-1.0.0        0403fd1c72c7        5 months ago        1.32GB
	hyperledger/fabric-couchdb     latest              2fbdbf3ab945        5 months ago        1.48GB
	hyperledger/fabric-couchdb     x86_64-1.0.0        2fbdbf3ab945        5 months ago        1.48GB
	hyperledger/fabric-kafka       latest              dbd3f94de4b5        5 months ago        1.3GB
	hyperledger/fabric-kafka       x86_64-1.0.0        dbd3f94de4b5        5 months ago        1.3GB
	hyperledger/fabric-zookeeper   latest              e545dbf1c6af        5 months ago        1.31GB
	hyperledger/fabric-zookeeper   x86_64-1.0.0        e545dbf1c6af        5 months ago        1.31GB
	hyperledger/fabric-orderer     latest              e317ca5638ba        5 months ago        179MB
	hyperledger/fabric-orderer     x86_64-1.0.0        e317ca5638ba        5 months ago        179MB
	hyperledger/fabric-peer        latest              6830dcd7b9b5        5 months ago        182MB
	hyperledger/fabric-peer        x86_64-1.0.0        6830dcd7b9b5        5 months ago        182MB
	hyperledger/fabric-javaenv     latest              8948126f0935        5 months ago        1.42GB
	hyperledger/fabric-javaenv     x86_64-1.0.0        8948126f0935        5 months ago        1.42GB
	hyperledger/fabric-ccenv       latest              7182c260a5ca        5 months ago        1.29GB
	hyperledger/fabric-ccenv       x86_64-1.0.0        7182c260a5ca        5 months ago        1.29GB
	hyperledger/fabric-ca          latest              a15c59ecda5b        5 months ago        238MB
	hyperledger/fabric-ca          x86_64-1.0.0        a15c59ecda5b        5 months ago        238MB
本次链码测试只用到了4个镜像：`fabric-tools、fabric-orderer、fabric-peer、fabric-ccenv`
## 3.启动网络
开启一个新的终端窗口，并在窗口中执行：

    docker-compose -f docker-compose-simple.yaml up
以上是使用`SingleSampleMSPSolo`orderer profile启动网络，并以“dev mode”启动peer。 它还启动了两个额外的容器 - 一个用于chaincode环境、一个使用CLI与chaincode交互。 创建和加入通道的命令被嵌入在CLI容器中，我们可以立即跳转到链码的调用。
## 4.编译并启动链码
开启一个新的终端窗口，并在窗口中执行：

    docker exec -it chaincode bash
进入到chaincode容器，然后编译链码，执行：
    
	cd sacc
    go build
之后运行链码，执行：
	
	CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc
请注意，在这个阶段链码不与任何通道相关联，需要在后续步骤中使用实例化命令完成。
## 5.使用链码
即使是使用`--peer-chaincodedev`模式，仍然需要安装链码，以便生命周期系统链接代码可以正常进行检查。我们将利用CLI容器进行这些命令的调用：

    docker exec -it cli bash
进入到CLI容器后，执行：

    peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
    peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc
现在发出一个调用将“a”的值更改为“20”：

	peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc
然后查询`a`的值，应该是`20`

	peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
