---
layout: post
title: "Fabric-SDK-GO通道实例"
date: 2020-09-10 
description: "使用SDK操作通道"
tag: 区块链 



---

------

# Fabric-SDK-GO

## 代码内容

本次程序主要是新建一个通道，然后在新建的通道上操作链码。以下为程序代码步骤：

1.  创建sdkClient
2.  通过sdkClient创建上下文
3.  构建资源管理器
4.  创建和更新通道
5.  节点加入通道
6.  创建通道管理器
7.  安装链码
8.  实例化链码
9.  升级链码
10.  调用链码
11.  链码查询

## 注意

新建通道需要给它指定通道配置，在`fabric-samples/first-network` 文件夹下执行命令：

`../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/通道配置名称.tx -channelID 通道名称`

​		因为是要使用sdk新建通道，如果新建的通道名也存在（如默认的`mychannel`），则会出现通道已存在相关的错误。我在每次执行后，都会修改通道名如（`mychannel1 2 3...`）。

![image-20200910104650042](http://hu12340.vip/images/posts\Fabric-SDK-GO\通道已存在.png)

​		如果学习过手动进行`fabrc-samples`网络部署，那么就会对**生成应用通道配置交易**这一步有印象。在新建通道时，需要指定通道配置。如自动化脚本中所使用的`mychannel`通道，它的的通道配置就是 `fabric-samples/first-network/channel-artifacts` 文件夹下的 `channel.tx`。如果没有生成对应的通道配置，则会出现找不到通道配置的错误。

![image-20200910105237733](http://hu12340.vip/images/posts\Fabric-SDK-GO\找不到通道配置.png)

​		即每次执行实例创建新通道时，都需要有它所对应的通道配置。那么如果我创建新通道让它的通道配置使用`mychannel`通道的`channel.tx`呢，如果这样的话就可以少了生成通道配置这一步了。结果是通道名不匹配，因为在生成通道配置时指定了通道的名称，如果创建通道的通道名称与指定的通道配置不匹配，则会出现以下错误。

![image-20200910110131936](http://hu12340.vip/images/posts\Fabric-SDK-GO\通道配置不匹配.png)

​		这时需要给新通道指定属于它的通道配置，在`fabric-samples/first-network` 文件夹下执行命令：

`../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/通道配置名称.tx -channelID 通道名称`  

​		新生成的通道配置会在`fabric-samples/first-network` 文件夹下（在命令中指定的文件夹），在代码中需要指定正确的通道配置及通道名称。在代码中指定了新生成的通道名及通道配置路径，并且也在对应的路径下生成了通道配置。还需要修改配置文件中的`channels`项的通道名称为新建通道的通道名称，否则在配置文件中，新建的通道会没有参与的节点，无法创建通道客户端。

![image-20200910112928555](http://hu12340.vip/images/posts\Fabric-SDK-GO\通道未指定节点.png)

​		这时需要修改配置文件中 `channels` 项里的通道名称为新建通道的名称。注意，这时通道已经创建成功，并且 `peer0.org1.example.com` 已成功加入通道。所以需要**重新创建通道以及生成新的通道配置，并且修改配置文件中的通道名称**。

![image-20200910113625611](http://hu12340.vip/images/posts\Fabric-SDK-GO\配置通道名称.png)

​		这时程序代码没有错误的话**应该**会正常运行。之所以说是应该，是因为现在的配置是一个概率问题。这个问题也是困扰了我很久，后来发现原因所在。在配置文件中的通道配置中，配置了`Org1`的两个节点 `peer0.org1.example.com` 、`peer1.org1.example.com` ；而我在代码中将节点加入通道时，只加入了 `peer0.org1.example.com` 。在创建通道客户端时，sdk会轮询配置文件中通道所配置的节点来创建通道客户端。由于配置了两个节点却只加入了一个节点，在轮询到已加入的节点并使用该节点创建通道时，创建通道会顺利执行；而如果轮询到的节点没有加入通道，在使用该节点创建通道客户端时就会出现没有权限的错误（因为该节点并没有加入通道，当然不能创建该通道的通道管理器）。

![image-20200910114430611](http://hu12340.vip/images/posts\Fabric-SDK-GO\没有权限.png)

​		这时有两种解决方式：一是将配置文件中配置了的节点都加入到该通道中；另一种方式比较简单，直接把没有加入通道的节点给注释掉。我在本实例中使用的是直接注释掉无用节点。

​		在程序执行成功后，可以进入容器使用 `peer channel list` 查看节点加入了哪些通道。因为使用全自动脚本启动网络，进入`cli`容器后，默认的节点就是 `peer0.org1.example.com`。

## 实例

### 配置文件

因为本实例中只使用了Org1，所以只需要组织1的配置文件。

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
      # 使用byfn中User1@org1的证书
      keyfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.key
      certfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.cert

# 如果应用程序已经创建了通道，则不需要这部分
channels:
  # 通道名称
  channel1:
    # 不要缺少当前channel的orderer节点
    orderers:
      - orderer.example.com
    #      - orderer2.example.com
    #      - orderer3.example.com
    #      - orderer4.example.com
    #      - orderer5.example.com
    peers:
    # 链码会使用以下配置的节点去连接通道，需要保证所配置的节点已加入了通道
      peer0.org1.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

       # 以下节点用于没有安装链码和加入通道，所以需要注释起来
#      peer1.org1.example.com:
#        endorsingPeer: true
#        chaincodeQuery: true
#        ledgerQuery: true
#        eventSource: true
#
#      peer0.org2.example.com:
#        endorsingPeer: true
#        chaincodeQuery: true
#        ledgerQuery: true
#        eventSource: true
#
#      peer1.org2.example.com:
#        endorsingPeer: true
#        chaincodeQuery: true
#        ledgerQuery: true
#        eventSource: true
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
    url: grpcs://192.168.102.179:7050
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
    # 表明使用grpcs协议，设置IP和端口号，使用域名会无法连接
    # url: grpcs://peer0.org1.example.com:7051
    url: grpcs://192.168.102.179:7051

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
    url: grpcs://192.168.102.179:8051
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
    url: grpcs://192.168.102.179:9051
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
    url: grpcs://192.168.102.179:10051
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
      urlsubstitutionexp: grpcs://192.168.102.179:7051
      ssltargetoverrideurlsubstitutionexp: peer0.org1.example.com
      mappedhost: peer0.org1.example.com

    - pattern: (\w*)peer1.org1.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.179:8051
      ssltargetoverrideurlsubstitutionexp: peer1.org1.example.com
      mappedhost: peer1.org1.example.com

    - pattern: (\w*)peer0.org2.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.179:9051
      ssltargetoverrideurlsubstitutionexp: peer0.org2.example.com
      mappedhost: peer0.org2.example.com

    - pattern: (\w*)peer1.org2.example.com(\w*)
      urlsubstitutionexp: grpcs://192.168.102.179:10051
      ssltargetoverrideurlsubstitutionexp: peer1.org2.example.com
      mappedhost: peer1.org2.example.com

  orderer:
    - pattern: (\w*)orderer.example.com(\w*)
      urlsubstitutionexp: 192.168.102.179:7050
      ssltargetoverrideurlsubstitutionexp: orderer.example.com
      mappedhost: orderer.example.com
```

### 主程序

本次实例也将程序所有代码放到了 `main.go` 中，在实际项目中需得注意代码分割。

``` yaml
package main

import (
	"errors"
	"github.com/hyperledger/fabric-sdk-go/pkg/client/channel"
	"github.com/hyperledger/fabric-sdk-go/pkg/client/resmgmt"
	"github.com/hyperledger/fabric-sdk-go/pkg/core/config"
	"github.com/hyperledger/fabric-sdk-go/pkg/fab/ccpackager/gopackager"
	"github.com/hyperledger/fabric-sdk-go/pkg/fab/resource"
	"github.com/hyperledger/fabric-sdk-go/pkg/fabsdk"
	"github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/common/cauthdsl"
	"log"
	"net/http"
	"os"
	"strings"
)

const (
	chaincodeName = "example1"
	chaincodePath = "github.com/hyperledger/fabric-samples/chaincode/chaincode_example02/go"
	version = "v1"
	upgradeVersion = "v2"
	peer0 = "peer0.org1.example.com"
)

func main(){
	////创建fabClient
	sdkClient,err := fabsdk.New(config.FromFile("config/org1sdk-config.yaml"))
	if err != nil {
		log.Panicf("创建SDK Client失败：%v \n",err)
	}
	//创建上下文
	resourceProvider := sdkClient.Context(fabsdk.WithUser("Admin"),fabsdk.WithOrg("Org1"))
	//构建资源管理器
	resourceClient,err := resmgmt.New(resourceProvider)
	if err != nil {
		log.Panicf("构建资源管理器失败：%v \n",err)
	}
	//创建或更新通道
	//通道配置和名称
	channelConfigPath := "/home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/channel-artifacts/" +
					"channel1.tx"
	channelName := "channel1"
	orderer := "orderer.example.com"
	resp, err := resourceClient.SaveChannel(
		resmgmt.SaveChannelRequest{
			ChannelID: channelName,
			ChannelConfigPath: channelConfigPath,
		},resmgmt.WithOrdererEndpoint(orderer))
	if err != nil {
		log.Panicf("创建或更新通道失败: %v\n", err)
	}
	if resp.TransactionID == "" {
		log.Println("创建或更新通道失败")
	}
	log.Printf("创建通道%s成功\n",channelName)
	//加入通道
	err = resourceClient.JoinChannel(channelName,resmgmt.WithTargetEndpoints("peer0.org1.example.com"))
	if err != nil {
		log.Panicf("加入通道失败：%v\n",err)
	}
	log.Println("peer0.org1.example.com加入通道成功")
	//创建通道管理器
	channelClient,err := channel.New(sdkClient.ChannelContext(channelName,fabsdk.WithUser("Admin"),fabsdk.WithOrg("Org1")))
	if err != nil {
		log.Panicf("创建通道客户端失败：%v\n",err)
	}
	log.Println("开始安装链码")
	//打包链码
	chaincodePKG,err := gopackager.NewCCPackage("github.com/hyperledger/fabric-samples/chaincode/chaincode_example02/go",os.Getenv("GOPATH"))
	if err != nil {
		log.Panicf("打包链码失败")
	}
	//安装链码
	installChaincode(resourceClient,chaincodeName,chaincodePath,version,chaincodePKG,peer0)
	//实例化链码
	//背书策略 AND表示需要里面的组织都进行背书 
	chaincodePolicy,err := cauthdsl.FromString("AND('Org1MSP.member','Org2MSP.member')")
	if err != nil {
		log.Panicf("构建背书策略失败:%v\n",err)
	}
	//打包参数
	//构建实例化链码请求
	requestOfInstantiate := resmgmt.InstantiateCCRequest{
		Name: chaincodeName,
		Path: chaincodePath,
		Version: version,
		Args: [][]byte{[]byte("init"),[]byte("a"),[]byte("100"),[]byte("b"),[]byte("200")},
		Policy: chaincodePolicy,
	}
	//目标节点
	responseOfInstantiete,err := resourceClient.InstantiateCC(channelName,requestOfInstantiate,resmgmt.WithTargetEndpoints(peer0))
	if err != nil {
		if strings.Contains(err.Error(),"already exists") {
			log.Println("链码已存在")
		}
		log.Panicf("实例化链码失败：%v\n",err)
	}
	log.Printf("链码实例化成功 交易ID：\n%v\n",responseOfInstantiete.TransactionID)
	//升级链码
	installChaincode(resourceClient,chaincodeName,chaincodePath,upgradeVersion,chaincodePKG,peer0)
	upgradePolicy,err := cauthdsl.FromString("OR('Org1MSP.member','Org2MSP.member')")
	if err != nil {
		log.Panicf("构建背书策略失败:%v\n",err)
	}
	//构建升级请求
	requestOfUpgrade := resmgmt.UpgradeCCRequest{
		Name: chaincodeName,
		Path: chaincodePath,
		Version: upgradeVersion,
		Args: [][]byte{[]byte("init"),[]byte("a"),[]byte("1000"),[]byte("b"),[]byte("2000")},
		Policy: upgradePolicy,
	}
	responseOfUpgrade,err := resourceClient.UpgradeCC(channelName,requestOfUpgrade,resmgmt.WithTargetEndpoints(peer0))
	if err != nil {
		log.Panicf("升级链码失败：%v\n",err)
	}
	log.Printf("升级链码交易ID：%v\n",responseOfUpgrade.TransactionID)
	//调用链码
	//构建调用请求
	requestOfInvoke := channel.Request{
		ChaincodeID: chaincodeName,
		Fcn: "invoke",
		Args: [][]byte{[]byte("a"),[]byte("b"),[]byte("10")},
	}
	//执行链码
	responseOfExecute,err := channelClient.Execute(requestOfInvoke,channel.WithTargetEndpoints(peer0))
	if err != nil {
		log.Panicf("链码执行失败：%v\n",err)
	}
	log.Printf("链码执行响应：\n" +
		"ID:%v\n" +
		"validate:%v\n" +
		"chaincode status:%v\n\n",
		responseOfExecute.TransactionID,
		responseOfExecute.TxValidationCode,
		responseOfExecute.ChaincodeStatus)
	//链码查询
	//构建查询请求
	requestOfQuery := channel.Request{
		ChaincodeID: chaincodeName,
		Fcn: "query",
		Args: [][]byte{[]byte("a")},
	}
	//执行链码
	responseOfQuery,err := channelClient.Query(requestOfQuery,channel.WithTargetEndpoints(peer0))
	if err != nil {
		log.Panicf("执行链码查询失败：%v\n",err)
	}
	log.Printf("链码查询结果：\n%s\n",string(responseOfQuery.Payload))
}

func installChaincode(resourceClient *resmgmt.Client,chaincodeName string, chaincodePath string, version string, chaincodePKG *resource.CCPackage, peer0 string) {
	//构建安装链码请求
	requestOfInstall := resmgmt.InstallCCRequest{
		Name: chaincodeName,
		Path: chaincodePath,
		Version: version,
		Package: chaincodePKG,
	}
	//安装链码
	responseOfInstall,err := resourceClient.InstallCC(requestOfInstall,resmgmt.WithTargetEndpoints(peer0))
	if err != nil {
		log.Panicf("安装链码失败:%v\n",err)
	}
	var errs []error
	for _,resp := range responseOfInstall {
		log.Printf("安装链码响应码：%v,\n",resp.Status)
		if resp.Status != http.StatusOK {
			errs = append(errs,errors.New(resp.Info))
		}
		if resp.Info == "already installed" {
			log.Printf("链码%s已安装在节点%s上\n",chaincodeName,resp.Target)
		}
	}
	if len(errs) > 0 {
		log.Panicf("安装链码失败%v\n",errs)
	}
	log.Println("安装链码完成")
}
```