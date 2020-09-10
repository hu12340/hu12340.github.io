---
layout: post
title: "客制化Fabric-samples"
date: 2020-08-26 
description: "自定义修改Fabric网络"
tag: 区块链 

---

------

# 概述

​		本次实例是在原生配置文件上做修改，将原来的两个组织（`Org1`、`Org2`）四个节点（`peer0.org1.example.com`、`peer1.org1.example.com`、`peer0.org2.example.com`、`peer1.org2.example.com`）修改为一个组织（`Yunphant`）三个节点（`peer0.yunphant.com`、`peer1.yunphant.com`、`peer2.yunphant.com`）。通过对配置文件的修改来深入了解各个配置属性的作用。

# 修改配置文件

​		为防止配置文件代码过长，将其中的空行和原有注释删除了，如果有兴趣的同学可以去拉取源码自行查看。本文中的这些配置文件只供参考，请勿直接复制粘贴（可能会因为误删了某些标点或空格造成配置文件出错）。

## configtx.yaml

该配置文件在`fabric-samples/first-network`目录下。删除原有`Org2`组织，将原有`Org1`组织改为`Yunphant`组织。

``` yaml
---
Organizations:  ## 定义组织
    - &OrdererOrg  ## Orderer组织
        Name: OrdererOrg  ## 名称
        ID: OrdererMSP  ## ID
        MSPDir: crypto-config/ordererOrganizations/example.com/msp  ## 证书、公私钥存储目录
        Policies:  ## 读写权限
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"
# 将Org1修改为Yunphant
    - &Yunphant
        # 修改名称
        Name: YunphantMSP
        # 修改ID
        ID: YunphantMSP
        # 修改MSP路径
        MSPDir: crypto-config/peerOrganizations/yunphant.com/msp
        # 修改对应权限
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('YunphantMSP.admin', 'YunphantMSP.peer', 'YunphantMSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('YunphantMSP.admin', 'YunphantMSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('YunphantMSP.admin')"
        AnchorPeers:
            # 修改锚节点
            - Host: peer0.yunphant.com  ## 主机地址
              Port: 7051  ## 端口
# 删除原有Org2组织
# 原有数据不做修改
Capabilities:  ##  版本信息
    Channel: &ChannelCapabilities
        V1_4_3: true
        V1_3: false
        V1_1: false
    # Orderer配置仅应用于排序节点，不需考虑对等节点的升级。将该配置项
    # 设置为true表明要求排序节点具备该能力
    Orderer: &OrdererCapabilities
        V1_4_2: true
        V1_1: false
    # Application配置仅应用于对等网络，不需考虑排序节点的升级。将该配置项
    # 设置为true表明要求对等节点具备该能力
    Application: &ApplicationCapabilities
        V1_4_2: true
        V1_3: false
        V1_2: false
        V1_1: false
Application: &ApplicationDefaults
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ApplicationCapabilities
Orderer: &OrdererDefaults  ##  Orderer切块策略
    OrdererType: solo  ##  共识算法
    Addresses:  ##  Order节点
        - orderer.example.com:7050
    BatchTimeout: 2s  ##  创建区块时超时时间 达到2s时创建新区块
    BatchSize:
        MaxMessageCount: 10  ##  交易达到10笔时创建新区块
        AbsoluteMaxBytes: 99 MB  ## 最大区块中交易总大小
        PreferredMaxBytes: 512 KB  ##  推荐区块中交易的大小
    Kafka:  ## 共识算法使用kafka时所使用的kafka地址
        Brokers:  ## 共识算法Raft
            - 127.0.0.1:9092
    EtcdRaft:
        Consenters:
            - Host: orderer.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
            - Host: orderer2.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
            - Host: orderer3.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
            - Host: orderer4.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
            - Host: orderer5.example.com
              Port: 7050
              ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
              ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
Channel: &ChannelDefaults
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ChannelCapabilities
Profiles:
    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                # 修改组织
                Organizations:
                    - *Yunphant
    TwoOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            # 修改组织
            Organizations:
                - *Yunphant
            Capabilities:
                <<: *ApplicationCapabilities
    SampleDevModeKafka:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Kafka:
                Brokers:
                - kafka.example.com:9092
            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                # 修改组织
                Organizations:
                - *Yunphant
    SampleMultiNodeEtcdRaft:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: etcdraft
            EtcdRaft:
                Consenters:
                - Host: orderer.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                - Host: orderer2.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                - Host: orderer3.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                - Host: orderer4.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                - Host: orderer5.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
            Addresses:
                - orderer.example.com:7050
                - orderer2.example.com:7050
                - orderer3.example.com:7050
                - orderer4.example.com:7050
                - orderer5.example.com:7050
            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                # 修改组织
                Organizations:
                - *Yunphant
```

## crypto-config.yaml

该配置文件在`fabric-samples/first-network`目录下。

``` yaml
# 排序节点
OrdererOrgs:
  - Name: Orderer  ## 名称
    Domain: example.com  ## 域名
    EnableNodeOUs: true  ## 如果设置了EnableNodeOUs，就在msp下生成config.yaml文件
    Specs:
      - Hostname: orderer
      - Hostname: orderer2
      - Hostname: orderer3
      - Hostname: orderer4
      - Hostname: orderer5
# Peer节点
PeerOrgs:
# 修改原有Org1配置为Yunphant配置
  - Name: Yunphant  ## 名称
    Domain: yunphant.com  ## 域名
    EnableNodeOUs: true  ## 如果设置了EnableNodeOUs，就在msp下生成config.yaml文件
    Template:
      Count: 3  ## Peer组织中Peer节点数量
    Users:
      Count: 1  ## 节点中普通用户的数量
```

## docker-compose-cli.yaml

该配置文件在`fabric-samples/first-network`目录下。

``` yaml
version: '2'
# 修改节点数据卷配置
volumes:
  orderer.example.com:
  peer0.yunphant.com:
  peer1.yunphant.com:
  peer2.yunphant.com:
networks:  ## 网络
  byfn:
services:
  orderer.example.com:
    extends:
      file:   base/docker-compose-base.yaml  ## 继承
      service: orderer.example.com
    container_name: orderer.example.com
    networks:
      - byfn
# 修改各个节点配置
  peer0.yunphant.com:
    container_name: peer0.yunphant.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.yunphant.com
    networks:
      - byfn
  peer1.yunphant.com:
    container_name: peer1.yunphant.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.yunphant.com
    networks:
      - byfn
  peer2.yunphant.com:
    container_name: peer2.yunphant.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer2.yunphant.com
    networks:
      - byfn
  cli:  ## 客户端
    container_name: cli
    image: hyperledger/fabric-tools:$IMAGE_TAG  ## 依赖镜像
    tty: true
    stdin_open: true
    environment:  ## 环境变量
      - SYS_CHANNEL=$SYS_CHANNEL
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      #- FABRIC_LOGGING_SPEC=DEBUG
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli
      # 指定cli容器默认节点
      - CORE_PEER_ADDRESS=peer0.yunphant.com:7051  ## 启动时默认链接
      # 指定cli容器默认组织
      - CORE_PEER_LOCALMSPID=YunphantMSP
      - CORE_PEER_TLS_ENABLED=true
      # 修改各种对应路径 证书 msp目录
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/peers/peer0.yunphant.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/peers/peer0.yunphant.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/peers/peer0.yunphant.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/users/Admin@yunphant.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:  ## 宿主机目录映射
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    # 修改节点
    depends_on:
      - orderer.example.com
      - peer0.yunphant.com
      - peer1.yunphant.com
      - peer2.yunphant.com

    networks:
      - byfn
```

## docker-compose-base.yaml

该配置文件在`fabric-samples/first-network/base`目录下。

``` yaml
version: '2'
services:
  ## order节点
  orderer.example.com:
    container_name: orderer.example.com
    extends:
      file: peer-base.yaml  ## 继承
      service: orderer-base
    volumes:  ## 挂载
        - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
        - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
        - orderer.example.com:/var/hyperledger/production/orderer
    ports:  ## 端口
      - 7050:7050
  # 修改节点配置
  peer0.yunphant.com:
    # 容器名称
    container_name: peer0.yunphant.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:  # 容器默认环境变量
      - CORE_PEER_ID=peer0.yunphant.com
      - CORE_PEER_ADDRESS=peer0.yunphant.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.yunphant.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.yunphant.com:8051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.yunphant.com:7051
      - CORE_PEER_LOCALMSPID=YunphantMSP
    volumes:  # 数据卷
        - /var/run/:/host/var/run/
        - ../crypto-config/peerOrganizations/yunphant.com/peers/peer0.yunphant.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/yunphant.com/peers/peer0.yunphant.com/tls:/etc/hyperledger/fabric/tls
        - peer0.yunphant.com:/var/hyperledger/production
    ports:  ## 对外端口
      - 7051:7051
  peer1.yunphant.com:
    container_name: peer1.yunphant.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer1.yunphant.com
      - CORE_PEER_ADDRESS=peer1.yunphant.com:8051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:8051
      - CORE_PEER_CHAINCODEADDRESS=peer1.yunphant.com:8052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:8052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.yunphant.com:8051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.yunphant.com:7051
      - CORE_PEER_LOCALMSPID=YunphantMSP
    volumes:
        - /var/run/:/host/var/run/
        - ../crypto-config/peerOrganizations/yunphant.com/peers/peer1.yunphant.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/yunphant.com/peers/peer1.yunphant.com/tls:/etc/hyperledger/fabric/tls
        - peer1.yunphant.com:/var/hyperledger/production
    ports:
      - 8051:8051
  peer2.yunphant.com:
    container_name: peer2.yunphant.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer2.yunphant.com
      - CORE_PEER_ADDRESS=peer2.yunphant.com:9051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
      - CORE_PEER_CHAINCODEADDRESS=peer2.yunphant.com:9052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2.yunphant.com:9051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.yunphant.com:7051
      - CORE_PEER_LOCALMSPID=YunphantMSP
    volumes:
        - /var/run/:/host/var/run/
        - ../crypto-config/peerOrganizations/yunphant.com/peers/peer2.yunphant.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/yunphant.com/peers/peer2.yunphant.com/tls:/etc/hyperledger/fabric/tls
        - peer2.yunphant.com:/var/hyperledger/production
    ports:
      - 9051:9051
```

# 配置网络环境

以下所涉及的命令都是在`fabric-samples/first-network`目录下进行执行。

## 生成证书和公私钥

`../bin/cryptogen generate --config=./crypto-config.yaml`

## 生成创世块

`../bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock channel-artifacts/genesis.block`

## 生成应用通道配置交易

`../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel`

## 更新锚节点

`../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/YunphantMSPPanchorstx -channelID mychannel -asOrg YunphantMSP`

## 启动容器

使用`docker-compose`启动容器

`docker-compose -f docker-compose-cli.yaml up -d`

# 进入容器

进入`cli`容器

`docker exec -it cli bash`

进入容器后默认默认目录为 `/opt/gopath/src/github.com/hyperledger/fabric/peer`

## 检查容器环境

### 通道

查看通道是否配置

`echo $CHANNEL_NAME`

不存在时设置

`export CHANNEL_NAME=mychannel`

### 证书

查看证书是否配置

`echo $ORDERER_CA`

不存在时设置

`export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`

### 创建通道

`peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls true --cafile $ORDERER_CA`

### 组织

查看当前组织

`echo $CORE_PEER_LOCALMSPID`

不存在时设置为当前组织

`export CORE_PEER_LOCALMSPID=YunphantMSP`

### MSP路径

查看当前节点MSP路径

`echo $CORE_PEER_MSPCONFIGPATH`

不存在时设置

`export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/users/Admin@yunphant.com/msp`

### 证书

查看当前节点安全证书

`echo $CORE_PEER_TLS_ROOTCERT_FILE`

不存在时设置为当前节点安全证书

`export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/peers/peer0.yunphant.com/tls/ca.crt`

### 节点

查看当前节点

`echo $CORE_PEER_ADDRSS`

不存在时配置为当前节点

`export CORE_PEER_ADDRSS=peer0.yunphant.com`

将当前节点设置加入通道

`peer channel join -b $CHANNEL_NAME.block`

查看通道

`peer channel list`

### 锚节点

更新锚节点

`peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/YunphantMSPPanchorstx --tls true --cafile $ORDERER_CA`

安装链码，进入`/opt/gopath/src`

 `peer chaincode install -n work2 -v 1.0 -p github.com/chaincode/work2/`

在当前节点实例化链码

`peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA -C mychannel -n work2 -v 1.0 -c '{"Args":["init"]}'`

在当前节点调用链码，插入数据

`peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n work2 -c '{"Args":["createProduct","1","pro","22"]}'`

切换节点

`export CORE_PEER_ADDRSS=peer1.yunphant.com`

`export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/peers/peer1.yunphant.com/tls/ca.crt`

调用链码查询

`peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n work2 -c '{"Args":["getProduct","1"]}'`

>   本实例使用的是单个组织，三个节点，并不需要更换组织
>
>   如需更换组织，还需配置组织名和对应的`msp`路径
>
>   `export CORE_PEER_LOCALMSPID="YunphantMSP"`
>
>   `export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/yunphant.com/users/Admin\@yunphant.com/msp/`
>
>   然后配置节点，加入通道安装链码等。
>
>   将当前节点设置加入通道，进入`/opt/gopath/src/github.com/hyperledger/fabric/peer`
>
>   `peer channel join -b $CHANNEL_NAME.block`
>
>   更新锚节点
>
>   `peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/YunphantMSPPanchorstx --tls true --cafile $ORDERER_CA`
>
>   安装链码，进入`/opt/gopath/src`
>
>    `peer chaincode install -n work2 -v 1.0 -p github.com/chaincode/work2/`

