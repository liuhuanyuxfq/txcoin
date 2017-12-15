
##1.修改docker-compose.yaml配置
###dc-orderer-kafka.yaml
	version: '2'
	
	networks:
	  byfn:
	
	services:
	  zookeeper0:
	    container_name: zookeeper0
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: zookeeper
	    environment:
	      - ZOO_MY_ID=1
	      - ZOO_SERVERS=server.1=zookeeper0:2888:3888 server.2=zookeeper1:2888:3888 server.3=zookeeper2:2888:3888
	    networks:
	      - byfn
	
	  zookeeper1:
	    container_name: zookeeper1
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: zookeeper
	    environment:
	      - ZOO_MY_ID=2
	      - ZOO_SERVERS=server.1=zookeeper0:2888:3888 server.2=zookeeper1:2888:3888 server.3=zookeeper2:2888:3888
	    networks:
	      - byfn
	
	  zookeeper2:
	    container_name: zookeeper2
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: zookeeper
	    environment:
	      - ZOO_MY_ID=3
	      - ZOO_SERVERS=server.1=zookeeper0:2888:3888 server.2=zookeeper1:2888:3888 server.3=zookeeper2:2888:3888
	    networks:
	      - byfn
	
	  kafka0:
	    container_name: kafka0
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: kafka
	    environment:
	      - KAFKA_BROKER_ID=0
	      - KAFKA_MIN_INSYNC_REPLICAS=2
	      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
	      - KAFKA_ZOOKEEPER_CONNECT=zookeeper0:2181,zookeeper1:2181,zookeeper2:2181
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	    ports:
	      - 9092:9092
	    networks:
	      - byfn
	
	  kafka1:
	    container_name: kafka1
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: kafka
	    environment:
	      - KAFKA_BROKER_ID=1
	      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
	      - KAFKA_MIN_INSYNC_REPLICAS=2
	      - KAFKA_ZOOKEEPER_CONNECT=zookeeper0:2181,zookeeper1:2181,zookeeper2:2181
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	    ports:
	      - 10092:9092
	    networks:
	      - byfn
	
	  kafka2:
	    container_name: kafka2
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: kafka
	    environment:
	      - KAFKA_BROKER_ID=2
	      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
	      - KAFKA_MIN_INSYNC_REPLICAS=2
	      - KAFKA_ZOOKEEPER_CONNECT=zookeeper0:2181,zookeeper1:2181,zookeeper2:2181
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	    ports:
	      - 11092:9092
	    networks:
	      - byfn
	
	  kafka3:
	    container_name: kafka3
	    extends:
	      file: base/dc-orderer-kafka-base.yaml
	      service: kafka
	    environment:
	      - KAFKA_BROKER_ID=3
	      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
	      - KAFKA_MIN_INSYNC_REPLICAS=2
	      - KAFKA_ZOOKEEPER_CONNECT=zookeeper0:2181,zookeeper1:2181,zookeeper2:2181
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	    ports:
	      - 12092:9092
	    networks:
	      - byfn
	
	  orderer0:
	    container_name: orderer0
	    extends:
	      file: base/dc-orderer-base.yaml
	      service: orderer
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	      - kafka0
	      - kafka1
	      - kafka2
	      - kafka3
	    ports:
	      - 7050:7050
	    networks:
	      - byfn
	
	  orderer1:
	    container_name: orderer1
	    extends:
	      file: base/dc-orderer-base.yaml
	      service: orderer
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	      - kafka0
	      - kafka1
	      - kafka2
	      - kafka3
	    ports:
	      - 8050:7050
	    networks:
	      - byfn
	
	  orderer2:
	    container_name: orderer2
	    extends:
	      file: base/dc-orderer-base.yaml
	      service: orderer
	    depends_on:
	      - zookeeper0
	      - zookeeper1
	      - zookeeper2
	      - kafka0
	      - kafka1
	      - kafka2
	      - kafka3
	    ports:
	      - 9050:7050
	    networks:
	      - byfn
###dc-orderer-base.yaml
	version: '2'
	
	services:
	
	  orderer:
	    image: hyperledger/fabric-orderer
	    environment:
	      - ORDERER_GENERAL_GENESISMETHOD=file
	      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
	      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
	      - ORDERER_GENERAL_LOGLEVEL=debug
	      - ORDERER_GENERAL_TLS_ENABLED=false
	      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
	      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
	      - ORDERER_KAFKA_VERBOSE=true
	      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
	      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
	    volumes:
	      - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
	      #- ./volumes/orderer:/var/hyperledger/bddtests/volumes/orderer
	      - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
	    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
	    command: orderer
###dc-orderer-kafka-base.yaml
	version: '2'
	
	services:
	
	  zookeeper:
	    image: hyperledger/fabric-zookeeper
	    restart: always
	    ports:
	      - '2181'
	      - '2888'
	      - '3888'
	
	  kafka:
	    image: hyperledger/fabric-kafka
	    restart: always
	    environment:
	      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
	      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
	      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
###注释掉docker-compose-cli.yaml中，orderer.example.com相关部分
##2.修改configtx.yaml
###将`Orderer:`部分做如下修改：
    OrdererType: kafka

    Addresses:
        - orderer0:7050
        - orderer1:8050
        - orderer2:9050
    Kafka:
        # Brokers: A list of Kafka brokers to which the orderer connects
        # NOTE: Use IP:port notation
        Brokers:
            - kafka0:9092
            - kafka1:10092
            - kafka2:11092
            - kafka3:12092

##3.启动网络
###(1)清理环境
    ./byfn.sh -m down
###(2)生成密码材料
    ./byfn.sh -m generate
	../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/achannel.tx -channelID achannel
###(3)启动网络
    docker-compose -f dc-orderer-kafka.yaml -f docker-compose-cli-kafka.yaml -f docker-compose-couch.yaml up -d
##4.创建通道、加入通道、部署链码、实例化链码、初始化链码，封装在kafka.sh中
    docker exec cli bash ./scripts/kafka.sh
##5.参考资源
[配置fabric-1.0的kafka模式](https://www.2cto.com/os/201708/671592.html)