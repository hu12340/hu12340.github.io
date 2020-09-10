---
layout: post
title: "Fabric-SDK-GO简单实例"
date: 2020-09-09 
description: "使用SDK操作链码"
tag: 区块链 


---

------

# Fabric-SDK-GO

​		Fabric-SDK-GO是Hyperledger Fabric官方提供的Go语言开发包，应用程序可以利用Fabric-SDK-GO与fabric网络进行交互并访问链码。简单的来说，就是用来**连接和操作Fabric网络**。

-   pkg/fabsdk：主package，主要用来生成fabsdk以及各种其他pkg使用的option context。
-   pkg/client/channel：主要用来调用、查询链码，或者注册链码事件。
-   pkg/client/resmgmt：主要用于fabric网络的管理，比如创建、加入通道，安装、实例化和升级链码。
-   pkg/client/event:配合channel模块来进行链码事件注册和过滤。
-   pkg/client/ledger：主要用于账本的查询，查询区块、交易、配置等。
-   pkg/client/msp：主要用来管理fabric的成员关系。

详细关于这些工具包的使用，可参阅[官方文档](https://godoc.org/github.com/hyperledger/fabric-sdk-go)。

## 使用步骤

1.  编写配置文件
2.  通过配置文件创建SDK Client
3.  通过SDK Client创建上下文
4.  通过上下文创建其它Client，如resource manage client、channel client。

## 使用实例

​		本实例连接的网络为 `fabric-samples` 中初始默认的 `byfn` 网络（一个 solo orderer，两个组织，四个节点）。如果要使用本实例，需先启动该网络。进入 `fabric-samples/frist-network` 目录，执行 `./byfn up` （可使用 `./byfn down` 关闭网络，脚本会自动清理所有数据）。本次实例使用的`fabric-samples`版本为`1.4.4`；为防止由于数据未清理干净或配置不同，可使用 `git status` 查看`fabric-samples`是否有修改并将其还原；使用 `docker ps` 及 `docker ps -a ` 查看是否还有fabri相关节点存在（一般在使用 `./byfn down` 后不会存在，若存在再次执行或手动关闭删除）；使用 `docker volume prune` 删除无用数据。

### go.mod

``` go
module sdk

go 1.14

require (
	github.com/Shopify/sarama v1.23.1 // indirect
	github.com/cloudflare/cfssl v0.0.0-20180323000720-5d63dbd981b5 // indirect
	github.com/fsouza/go-dockerclient v1.4.4 // indirect
	github.com/grpc-ecosystem/go-grpc-middleware v1.0.0 // indirect
	github.com/hashicorp/go-version v1.2.0 // indirect
	github.com/hyperledger/fabric v1.4.3
	github.com/hyperledger/fabric-amcl v0.0.0-20181230093703-5ccba6eab8d6 // indirect
	github.com/hyperledger/fabric-sdk-go v1.0.0-alpha5.0.20190411180201-5a9a0e749e4f
	github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric v0.0.0-20190411180201-5a9a0e749e4f
	github.com/magiconair/properties v1.8.0 // indirect
	github.com/mitchellh/mapstructure v0.0.0-20180511142126-bb74f1db0675 // indirect
	github.com/op/go-logging v0.0.0-20160315200505-970db520ece7 // indirect
	github.com/pelletier/go-toml v1.2.0 // indirect
	github.com/pkg/errors v0.8.1
	github.com/prometheus/common v0.0.0-20180801064454-c7de2306084e // indirect
	github.com/prometheus/procfs v0.0.0-20180920065004-418d78d0b9a7 // indirect
	github.com/spf13/afero v1.1.1 // indirect
	github.com/stretchr/objx v0.2.0 // indirect
	github.com/sykesm/zap-logfmt v0.0.2 // indirect
	go.uber.org/zap v1.10.0 // indirect
	gopkg.in/jcmturner/goidentity.v3 v3.0.0 // indirect
)
```

### 配置文件

​		配置文件中的IP地址需更换为自己`byfn`网络启动所在的主机IP地址，并且需保证端口可用。配置文件中指定的 `crypto-config` 位置，需指定为`byfn`网络启动后生成的`crypto-config`文件夹。如我在`fabric-samples/frist-network` 目录执行 `./byfn up` ，则会在该目录下生成一个 `crypto-config` 文件夹。将配置文件中关于 `crypto-config` 文件夹的配置都指向启动网络所生成的 `crypto-config` 文件夹。

#### org1sdk-config.yaml

``` yaml
version: 1.0.0
#定义 SDK 客户端
client:
  # 客户端所属组织，必须是organization定义的组织
  organization: Org1
  logging:
  # 打印日志等级
    level: info
  # MSP根目录 链接byfn网络时，指定为网络启动后生成的crypto-config文件夹
  cryptoconfig:
    path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config
  # 某些SDK支持插件化的KV数据库，通过该属性实现
  credentialStore:
    # 可选，用于用户证书材料存储，如果所有的证书材料被嵌入到配置文件，则不需要
    path: "/tmp/examplestore"
  # 客户端的BCCSP模块配置
  BCCSP:
    security:
      enabled: true
      default:
        provider: "SW"
      hashAlgorithm: "SHA2"
      softVerify: true
      level: 256

  tlsCerts:
    # 可选，当连接到peers，orderers时使用系统证书池，默认为false
    systemCertPool: true
    #  可选，客户端和peers与orderers进行TLS握手的密钥和证书
    client:
      # 使用byfn中User1@org1的证书
      keyfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.key
      certfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.cert

# 如果应用程序已经创建了通道，则不需要这部分
channels:
  # 通道名称
  mychannel:
    # 不要缺少当前channel的orderer节点
    orderers:
      - orderer.example.com
    peers:
      peer0.org1.example.com:
        #参与背书
        endorsingPeer: true
        # 调用链码查询
        chaincodeQuery: true
        # 查询账本
        ledgerQuery: true
        # 监听事件
        eventSource: true

      peer1.org1.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      peer0.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      peer1.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

    # 可选，应用程序使用下列选项执行通道操作，如获取通道配置
    policies:
      # 可选，获取通道配置
      queryChannelConfig:
        #成功响应节点的最小数量
        minResponses: 1
        maxTargets: 1
        # 查询配置区块的重试选项
        retryOpts:
          # 重试次数
          attempts: 5
          initialBackoff: 500ms
          #第一次重试的后退间隔
          maxBackoff: 5s

          backoffFactor: 2.0
# Fabric区块链网络中参与的组织列表
organizations:
 # 组织名
  Org1:
    mspid: Org1MSP
    # 组织的MSP存储位置，绝对路径或相对cryptoconfig的路径
    cryptoPath: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/{username}@org1.example.com/msp

    # 组织中的节点
    peers:
      - peer0.org1.example.com
      - peer1.org1.example.com

  Org2:
    mspid: Org2MSP
    cryptoPath: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/{username}@org2.example.com/msp

    peers:
      - peer0.org2.example.com
      - peer1.org2.example.com

  ordererorg:
    mspID: OrdererMSP
    cryptoPath: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp

# 发送交易请求或通道创建、更新请求到的orderers列表
# 如果定义了超过一个orderer，SDK使用哪一个orderer由代码实现时指定
orderers:
  orderer.example.com:
    url: grpcs://192.168.102.146:7050

    # 以下属性由gRPC库定义，会被传递给gRPC客户端构造函数
    grpcOptions:
      # 下列参数用于设置服务器上的keepalive策略，不兼容的设置会导致连接关闭
      # 当keep-alive-time被设置为0或小于激活客户端的参数，下列参数失效
      ssl-target-name-override: orderer.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      allow-insecure: false

    tlsCACerts:
      #证书的绝对路径
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
#peers必须指定Hyperledger Fabric区块链网络中所有peer节点的主机名和端口，可能会在其它地方引用，如channels，organizations等部分。
peers:
  peer0.org1.example.com:
    # 表明使用grpcs协议，设置IP和端口号，使用域名会无法连接
    # url: grpcs://peer0.org1.example.com:7051
    url: grpcs://192.168.102.146:7051

    grpcOptions:
      ssl-target-name-override: peer0.org1.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      allow-insecure: false
    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem

  peer1.org1.example.com:
    # 表明使用grpcs协议，设置IP和端口号，使用域名会无法连接
    # url: grpcs://peer0.org1.example.com:7051
    url: grpcs://192.168.102.146:8051
    grpcOptions:
      ssl-target-name-override: peer1.org1.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      allow-insecure: false

    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem

  peer0.org2.example.com:
    url: grpcs://192.168.102.146:9051
    grpcOptions:
      ssl-target-name-override: peer0.org2.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false

      allow-insecure: false
    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem
  peer1.org2.example.com:
    url: grpcs://192.168.102.146:10051
    grpcOptions:
      ssl-target-name-override: peer1.org2.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false

      allow-insecure: false
    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem

entitymatchers:
  peer:
    - pattern: (\w*)peer0.org1.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:7051
      ssltargetoverrideurlsubstitutionexp: peer0.org1.example.com
      mappedhost: peer0.org1.example.com

    - pattern: (\w*)peer1.org1.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:8051
      ssltargetoverrideurlsubstitutionexp: peer1.org1.example.com
      mappedhost: peer1.org1.example.com

    - pattern: (\w*)peer0.org2.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:9051
      ssltargetoverrideurlsubstitutionexp: peer0.org2.example.com
      mappedhost: peer0.org2.example.com

    - pattern: (\w*)peer1.org2.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:10051
      ssltargetoverrideurlsubstitutionexp: peer1.org2.example.com
      mappedhost: peer1.org2.example.com

  orderer:
    - pattern: (\w*)orderer.example.com(\w*)
      urlsubstitutionexp: 192.168.102.146:7050
      ssltargetoverrideurlsubstitutionexp: orderer.example.com
      mappedhost: orderer.example.com
```

#### org2sdk-config.yaml

``` yaml
version: 1.0.0
client:
  organization: Org1
  logging:
    level: info
  cryptoconfig:
    path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config

  credentialStore:
    path: "/tmp/examplestore"

  BCCSP:
    security:
      enabled: true
      default:
        provider: "SW"
      hashAlgorithm: "SHA2"
      softVerify: true
      level: 256

  tlsCerts:
    systemCertPool: true
    client:
      keyfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/User1@org2.example.com/tls/client.key
      certfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/User1@org2.example.com/tls/client.cert

channels:
  mychannel:
    orderers:
      - orderer.example.com
    peers:
      peer0.org1.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      peer1.org1.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      peer0.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      peer1.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

    policies:
      queryChannelConfig:
        minResponses: 1
        maxTargets: 1
        retryOpts:
          attempts: 5
          initialBackoff: 500ms
          maxBackoff: 5s
          backoffFactor: 2.0

organizations:
  Org1:
    mspid: Org1MSP
    cryptoPath: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/{username}@org1.example.com/msp

    peers:
      - peer0.org1.example.com
      - peer1.org1.example.com

  Org2:
    mspid: Org2MSP
    cryptoPath: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/{username}@org2.example.com/msp

    peers:
      - peer0.org2.example.com
      - peer1.org2.example.com

  ordererorg:
    mspID: OrdererMSP
    cryptoPath: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp

orderers:
  orderer.example.com:
    url: grpcs://192.168.102.146:7050
    grpcOptions:
      ssl-target-name-override: orderer.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      allow-insecure: false

    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem

peers:
  peer0.org1.example.com:
    url: grpcs://192.168.102.146:7051

    grpcOptions:
      ssl-target-name-override: peer0.org1.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      allow-insecure: false

    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem

  peer1.org1.example.com:
    url: grpcs://192.168.102.146:8051
    grpcOptions:
      ssl-target-name-override: peer1.org1.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false

      allow-insecure: false

    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem

  peer0.org2.example.com:

    url: grpcs://192.168.102.146:9051
    grpcOptions:
      ssl-target-name-override: peer0.org2.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false

      allow-insecure: false
    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem

  peer1.org2.example.com:
    url: grpcs://192.168.102.146:10051
    grpcOptions:
      ssl-target-name-override: peer1.org2.example.com
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false

      allow-insecure: false
    tlsCACerts:
      path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem

entitymatchers:
  peer:
    - pattern: (\w*)peer0.org1.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:7051
      ssltargetoverrideurlsubstitutionexp: peer0.org1.example.com
      mappedhost: peer0.org1.example.com

    - pattern: (\w*)peer1.org1.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:8051
      ssltargetoverrideurlsubstitutionexp: peer1.org1.example.com
      mappedhost: peer1.org1.example.com

    - pattern: (\w*)peer0.org2.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:9051
      ssltargetoverrideurlsubstitutionexp: peer0.org2.example.com
      mappedhost: peer0.org2.example.com

    - pattern: (\w*)peer1.org2.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.146:10051
      ssltargetoverrideurlsubstitutionexp: peer1.org2.example.com
      mappedhost: peer1.org2.example.com

  orderer:
    - pattern: (\w*)orderer.example.com(\w*)
      urlsubstitutionexp: 192.168.102.146:7050
      ssltargetoverrideurlsubstitutionexp: orderer.example.com
      mappedhost: orderer.example.com
```

### main.go

``` go
package main

import (
	"github.com/hyperledger/fabric-sdk-go/pkg/client/channel"
	"github.com/hyperledger/fabric-sdk-go/pkg/client/resmgmt"
	"github.com/hyperledger/fabric-sdk-go/pkg/common/providers/fab"
	"github.com/hyperledger/fabric-sdk-go/pkg/core/config"
	"github.com/hyperledger/fabric-sdk-go/pkg/fab/ccpackager/gopackager"
	"github.com/hyperledger/fabric-sdk-go/pkg/fabsdk"
	"github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/common/cauthdsl"
	"github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/protos/common"
	"github.com/pkg/errors"
	"log"
	"net/http"
	"os"
	"strings"
)

const (
	org1Config = "config/org1sdk-config.yaml"
	org2Config = "config/org2sdk-config.yaml"
	org1 = "Org1"
	org2 = "Org2"
	admin = "Admin"
	user = "User1"
	chaincodePath = "github.com/hyperledger/fabric-samples/chaincode/chaincode_example02/go"
)

var (
	peer0Org1 = "peer0.org1.example.com"
	peer0Org2 = "peer0.org2.example.com"
)

func main()	{
	org1Client := NewClient(org1Config,org1,admin,user)
	org2Client := NewClient(org2Config,org2,admin,user)

	// Close: 关闭并释放有SDK维护的缓存和连接
	defer org1Client.SDK.Close()
	defer org2Client.SDK.Close()

	InstallAndInvoke(org1Client,org2Client,"v1")
	InstallAndUpgrade(org1Client,org2Client,"v2")
}

func InstallAndUpgrade(client1 *Client, client2 *Client, version string) {
	log.Println("------------------------------------安装并升级 开始------------------------------")
	defer log.Println("------------------------------------安装并升级 结束------------------------------")
	if err := client1.InstallChaincode(version,peer0Org1); err != nil {
		log.Panicf("安装链码失败:%v",err)
	}
	log.Println("链码已成功安装到Org1的节点上")
	if err := client2.InstallChaincode(version,peer0Org2); err != nil {
		log.Panicf("安装链码失败:%v",err)
	}
	log.Println("链码已成功安装到Org2的节点上")
	if err := client1.UpgradeChaincode(version,peer0Org1); err != nil {
		log.Panicf("链码升级失败:%v",err)
	}
	if _,err := client1.InvokeChaincode([]string{peer0Org1,peer0Org2});err != nil {
		log.Panicf("链码调用失败：%v",err)
	}
	log.Println("链码调用成功")
	if err := client1.QueryChaincode(peer0Org2,"a"); err != nil {
		log.Panicf("链码查询失败：%v",err)
	}
	log.Println("在peer0.org2上使用链码查询成功")
}

func  InstallAndInvoke(client1,client2 *Client,version string) {
	log.Println("------------------------------------安装并实例化 开始------------------------------")
	defer log.Println("------------------------------------安装并实例化 结束------------------------------")
	if err := client1.InstallChaincode(version,peer0Org1);err != nil {
		log.Panicf("安装链码失败:%v",err)
	}
	log.Println("链码已成功安装到组织1上")
	if err := client2.InstallChaincode(version,peer0Org2);err != nil {
		log.Panicf("安装链码失败:%v",err)
	}
	log.Println("链码已成功安装到组织2上")

	if _,err := client1.InstantiateChaincode(version,peer0Org1);err != nil {
		log.Panicf("实例化链码失败：%v",err)
	}
	log.Println("链码已成功实例化")

	if _,err := client1.InvokeChaincode([]string{peer0Org1}); err != nil {
		log.Printf("调用链码失败:%v",err)
	}
	log.Println("调用链码成功")

	if err := client1.QueryChaincode(peer0Org1,"a"); err != nil {
		log.Panicf("链码查询失败:%v",err)
	}
	log.Println("成功在peer0.org1上使用链码查询")

}

func (client *Client) QueryChaincode( peer string, keys string) error {
	//构建查询请求 Fcn：自己链码中invoke函数里的case选项，并非调用类型
	req := channel.Request{
		ChaincodeID: client.ChaincodeID,
		Fcn: "query",
		Args: packArgs([]string{keys}),
	}
	//目标节点
	reqPeers := channel.WithTargetEndpoints(peer)
	//client.Query: 根据请求可选项查询链码
	resp,err := client.cc.Query(req,reqPeers)
	if err != nil {
		return errors.WithMessage(err,"链码查询失败")
	}
	log.Printf("链码查询 tx 响应：\ntx:%s\nresult:%v\n\n",resp.TransactionID,string(resp.Payload))
	return nil
}

func (client *Client) InvokeChaincode(peers []string) (fab.TransactionID, error) {
	args := packArgs([]string{"a", "b", "10"})
	//构建查询请求 Fcn：自己链码中invoke函数里的case选项，并非调用类型
	req := channel.Request{
		ChaincodeID: client.ChaincodeID,
		Fcn: "invoke",
		Args: args,
	}
	reqPeers := channel.WithTargetEndpoints(peers...)
	//client.Execute: 根据请求和请求可选项调用链码
	resp,err := client.cc.Execute(req,reqPeers)
	log.Printf("调用链码响应：\n" +
		"id：%v\n" +
		"validate:%v\n" +
		"chaincode status:%v\n\n",
		resp.TransactionID,
		resp.TxValidationCode,
		resp.ChaincodeStatus)
	if err != nil {
		return "",errors.WithMessage(err,"调用链码失败")
	}
	return resp.TransactionID,nil
}

func (client *Client) InstantiateChaincode(version string, peer string) (fab.TransactionID, error) {
	//背书策略
	org1OrOrg2 := "OR('Org1MSP.member','Org2MSP.member')"
	chaincodePolicy,err := client.genPolicy(org1OrOrg2)
	if err != nil {
		return "",errors.WithMessage(err,"gen policy from string error")
	}
	args := packArgs([]string{"init", "a", "100", "b", "200"})
	req := resmgmt.InstantiateCCRequest{
		Name: client.ChaincodeID,
		Path: client.ChaincodePath,
		Version: version,
		Args: args,
		Policy: chaincodePolicy,
	}
	reqPeers := resmgmt.WithTargetEndpoints(peer)
	//resmgmt.InstantiateCC: 实例化链码
	resp,err := client.rc.InstantiateCC(client.ChannelID,req,reqPeers)
	if err != nil {
		if strings.Contains(err.Error(),"already exists"){
			return "",nil
		}
		return "",errors.WithMessage(err,"实例化链码失败")
	}
	log.Printf("实例化链码 tx:%v",resp.TransactionID)
	return resp.TransactionID,nil
}

func packArgs(paras []string) [][]byte{
	var args [][]byte
	for _,k := range paras {
		args = append(args,[]byte(k))
	}
	return args
}

func (client *Client) genPolicy(p string) (*common.SignaturePolicyEnvelope,error){
	if p == "ANY" {
		return cauthdsl.SignedByAnyMember([]string{client.OrgName}),nil
	}
	return cauthdsl.FromString(p)
}

func (client *Client) InstallChaincode(version string, peer string) error {
	targetPeer := resmgmt.WithTargetEndpoints(peer)
	//打包链码
	chaincodePKG,err := gopackager.NewCCPackage(client.ChaincodePath,client.GoPath)
	if err != nil {
		return errors.WithMessage(err,"打包链码失败")
	}

	req := resmgmt.InstallCCRequest{
		Name: client.ChaincodeID,
		Path: client.ChaincodePath,
		Version: version,
		Package: chaincodePKG,
	}
	//resmgmt.InstallCC: 安装链码
	resps,err := client.rc.InstallCC(req,targetPeer)
	if err != nil {
		return errors.WithMessage(err,"安装链码失败")
	}

	var errs []error
	for _,resp := range resps {
		log.Printf("安装响应码：%v",resp.Status)
		if resp.Status != http.StatusOK {
			errs = append(errs,errors.New(resp.Info))
		}
		if resp.Info == "already installed" {
			log.Printf("链码 %s 已安装在节点：%s.\n",client.ChaincodeID,resp.Target)
			return nil
		}
	}
	if len(errs) > 0 {
		log.Panicf("安装链码失败：%v",errs)
		return errors.WithMessage(errs[0],"安装链码首次失败位置")
	}
	return nil
}

func (client *Client) UpgradeChaincode(version string, peer string) error {
	org1AndOrg2 := "AND('Org1MSP.member','Org2MSP.member')"
	chaincodePolicy,err := client.genPolicy(org1AndOrg2)
	if err != nil {
		return errors.WithMessage(err,"gen policy from string error")
	}
	args := packArgs([]string{"init", "a", "1000", "b", "2000"})
	req := resmgmt.UpgradeCCRequest{
		Name: client.ChaincodeID,
		Path: client.ChaincodePath,
		Version: version,
		Args: args,
		Policy: chaincodePolicy,
	}
	reqPerrs := resmgmt.WithTargetEndpoints(peer)
	//resmgmt.UpgradeCC: 升级链码
	resp,err := client.rc.UpgradeCC(client.ChannelID,req,reqPerrs)
	if err != nil {
		return errors.WithMessage(err,"实例化链码失败")
	}
	log.Printf("实例化链码 tx ：%v",resp.TransactionID)
	return nil
}

func NewClient(cfg, org, admin, user string) *Client{
	client := &Client{
		ConfigPath: cfg,
		OrgName: org,
		OrgAdmin: admin,
		OrgUser: user,

		ChaincodeID: "example",
		ChaincodePath: chaincodePath,
		GoPath: os.Getenv("GOPATH"),
		ChannelID: "mychannel",
	}
	// fabsdk.New：创建fabsdk实例
	sdk,err := fabsdk.New(config.FromFile(client.ConfigPath))
	if err != nil {
		log.Panicf("创建SDK失败：%s",err)
	}
	client.SDK = sdk
	log.Println("SDK初始化完成")
	client.rc, client.cc = NewSDKClient(sdk, client.ChannelID, client.OrgName, client.OrgAdmin, client.OrgUser)
	return client
}

func NewSDKClient(sdk *fabsdk.FabricSDK, channelID string, orgName string, orgAdmin string, orgUser string) (rc *resmgmt.Client, cc *channel.Client) {
	var err error

	//WithIdentity：使用预先构造的身份对象作为访问的凭据
	//WithOrg：使用命名的组织
	//WithUser：使用命名的用户来加载对应的身份证书
	//Context：创建并返回上下文客户端
	rcp := sdk.Context(fabsdk.WithUser(orgAdmin),fabsdk.WithOrg(orgName))
	// resmgmt.New: 返回一个client实例
	rc,err = resmgmt.New(rcp)
	if err != nil {
		log.Panicf("创建RC客户端失败:%s",err)
	}
	log.Println("RC初始化完成")
	//ChannelContext：创建并返回通道上下文
	ccp := sdk.ChannelContext(channelID,fabsdk.WithUser(orgUser))
	//channel.New: 返回一个Client实例
	cc,err = channel.New(ccp)
	if err != nil {
		log.Panicf("创建通道客户端失败：%s",err)
	}
	log.Println("通道客户端初始化完成")
	return rc,cc
}

type Client struct {
	// Fabric 网络信息
	ConfigPath string
	OrgName string
	OrgAdmin string
	OrgUser string

	//sdk 客户端
	SDK *fabsdk.FabricSDK
	rc *resmgmt.Client
	cc *channel.Client

	// 链码信息
	ChannelID string //通道名
	ChaincodeID string //链码ID或者名称
	ChaincodePath string //链码路径
	GoPath string // GOPATH路径
}
```

### 测试

进入链码路径，执行 `go run main.go` 。

打印日志

```
 [fabsdk/fab] 2020/09/09 02:31:25 UTC - fab.detectDeprecatedNetworkConfig -> WARN Getting orderers from endpoint config channels.orderer is deprecated, use entity matchers to override orderer configuration
 [fabsdk/fab] 2020/09/09 02:31:25 UTC - fab.detectDeprecatedNetworkConfig -> WARN visit https://github.com/hyperledger/fabric-sdk-go/blob/master/test/fixtures/config/overrides/local_entity_matchers.yaml for samples
2020/09/09 10:31:25 SDK初始化完成
2020/09/09 10:31:25 RC初始化完成
2020/09/09 10:31:25 通道客户端初始化完成
 [fabsdk/fab] 2020/09/09 02:31:25 UTC - fab.detectDeprecatedNetworkConfig -> WARN Getting orderers from endpoint config channels.orderer is deprecated, use entity matchers to override orderer configuration
 [fabsdk/fab] 2020/09/09 02:31:25 UTC - fab.detectDeprecatedNetworkConfig -> WARN visit https://github.com/hyperledger/fabric-sdk-go/blob/master/test/fixtures/config/overrides/local_entity_matchers.yaml for samples
2020/09/09 10:31:25 SDK初始化完成
2020/09/09 10:31:25 RC初始化完成
2020/09/09 10:31:25 通道客户端初始化完成
2020/09/09 10:31:25 ------------------------------------安装并实例化 开始------------------------------
2020/09/09 10:31:25 安装响应码：200
2020/09/09 10:31:25 链码已成功安装到组织1上
2020/09/09 10:31:25 安装响应码：200
2020/09/09 10:31:25 链码已成功安装到组织2上
2020/09/09 10:31:25 链码已成功实例化
2020/09/09 10:31:28 调用链码响应：
id：f6a125f4f0d7acf5580c08e3310b13c79be8aa62e2aecb2dfb4106b838c2f530
validate:VALID
chaincode status:200

2020/09/09 10:31:28 调用链码成功
2020/09/09 10:31:28 链码查询 tx 响应：
tx:0c0ae2993a62a4386c0402980e653b4aebccfb87dd9a027b7592ed65097f884f
result:80

2020/09/09 10:31:28 成功在peer0.org1上使用链码查询
2020/09/09 10:31:28 ------------------------------------安装并实例化 结束------------------------------
2020/09/09 10:31:28 ------------------------------------安装并升级 开始------------------------------
2020/09/09 10:31:28 安装响应码：200
2020/09/09 10:31:28 链码已成功安装到Org1的节点上
2020/09/09 10:31:28 安装响应码：200
2020/09/09 10:31:28 链码已成功安装到Org2的节点上
2020/09/09 10:33:06 实例化链码 tx ：eff15eba14cae1304ebb0b38ddacb33fa67a1b26f05c1a1756c8d25ff990a04b
2020/09/09 10:34:01 调用链码响应：
id：b16a412fd63788a8085ce5b63c37c4e062ef7d16a2ba2f128f0eef31a91591d5
validate:VALID
chaincode status:200

2020/09/09 10:34:01 链码调用成功
2020/09/09 10:34:01 链码查询 tx 响应：
tx:f18bd37b1450b95ba4d881a7966245be4abc50d3a2d52030fd1685b42e6dd882
result:1000

2020/09/09 10:34:01 在peer0.org2上使用链码查询成功
2020/09/09 10:34:01 ------------------------------------安装并升级 结束------------------------------
```

注意：

-   记得修改链码路径以及网络地址
-   如果链码已安装到组织中，再次安装则会失败
-   多次执行该实例时，记得修改链码版本

## 心得

​		对于Fabric-SDK-GO，在拉取完源码后，发现文件夹就有很多，不知道怎么使用。里面的test项目，如e2e一直看不懂，也跑不起来。后来看到了[大彬的博客](https://lessisbetter.site/)以及[SDK入门](https://lessisbetter.site/2019/09/02/fabric-sdk-go-chaincode/)，开始学习[大彬写的sdk项目](https://github.com/Shitaibin/fabric-sdk-go-sample)。其实Fabric-SDK-GO和Fabric差不多，拉取到源码后都有很多文件夹和工具包，但其实这些都没有用。拉取下来可能也就是查看里面的操作实例以及源码，在项目使用时也就是导入它们的源码包。

​		在拉取到大彬的sdk-sample项目后，发现他所构建的项目也都是很规范的。不同功能的工具代码在不同的go文件中，也根据模块进行文件夹分隔。但对于一个刚入手的我来说，这些规范就让我觉得果然源码包中那么多的文件夹都是需要用到的。在认真学习了大彬的sdk项目之后，发现各个go文件也就只是在做功能分割，sdk也只是导入当做工具包使用。

​		大彬的sdk项目可能是因为自己需要做调试，有些初始源码被修改然后推送，自己在照着他的sdk项目学习时，也发现了这个问题。别人的源码项目并不是让我们一味的照搬，更多的是去理解这些代码及配置的作用以及正确使用。

>   比如在大彬的SDK项目中，一开始定义了一个Client（自定义结构体，类似于java中的类，方便使用。并不是源码中的结构体）并指定了一些基础属性，如链码名称（CCID）。在他的项目中的有些地方使用的是Client.CCID，有些地方又是使用的字符串（mycc）；他在构建结构体时所指定的CCID为 example4 ，而在后面使用的中有有用到 mycc 作为链码名，在运行时可能就会造成错误。我看过他的git修改记录，原本都是全部使用的Client.CCID，后面有些地方修改为了 mycc ，应该是在做调试并且push了。当然，大彬的项目只是让我们做参考，更多的是知道sdk的各种工具包以及方法接口的使用。

​		为了让项目看起来简单，我把代码都放到了main.go中，只是为了让刚入手的小白看起来更好理解。这样看起来写代码没有条理，后期回顾时如果没有很好的注释，看起来会很难。在实际的项目中，需要做到代码规范。

​		另外，在整理学习文档时，尽量在当天就及时整理，即使任务并没有做完。因为在之后可能会忘记或者忘掉当时的众多感受。