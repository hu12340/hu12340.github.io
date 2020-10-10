# Fabric-SDK-GO

本文主要参考[《如何在Fabric SDK Go中添加集合配置？》](https://ask.csdn.net/questions/1021515?sort=votes_count)，博客底部提供的解答实测有效。

​		在之前学习Fabric链码时，就接触到了私密数据。当时使用私密数据是在节点容器中，实例化链码时指定的集合配置（`--collections-config=./私密数据文件`）。在sdk-go中也是需要在链码实例化的时候指定集合配置。

## 构建方法

以下为构建集合配置的主要方法：

``` go
func newCollectionConfig(colName, policy string, reqPeerCount, maxPeerCount int32, blockToLive uint64) (*common.CollectionConfig, error) {
	p, err := cauthdsl.FromString(policy)
	if err != nil {
		return nil, err
	}
	cpc := &common.CollectionPolicyConfig{
		Payload: &common.CollectionPolicyConfig_SignaturePolicy{
			SignaturePolicy: p,
		},
	}
	return &common.CollectionConfig{
		Payload: &common.CollectionConfig_StaticCollectionConfig{
			StaticCollectionConfig: &common.StaticCollectionConfig{
				Name:              colName,
				MemberOrgsPolicy:  cpc,
				RequiredPeerCount: reqPeerCount,
				MaximumPeerCount:  maxPeerCount,
				BlockToLive:       blockToLive,
			},
		},
	}, nil
}
```

## 实例

​		由于本次实例涉及到的代码比较多，请直接查看实例链码部分如何加入集合配置。其实很简单，在构建实例化请求时指定`CollConfig`属性。

​		**本次实例代码过多过长，实例部分并不建议观看，可直接去[《如何在Fabric SDK Go中添加集合配置？》](https://ask.csdn.net/questions/1021515?sort=votes_count)查看。**

​		**本次实例代码过多过长，实例部分并不建议观看，可直接去[《如何在Fabric SDK Go中添加集合配置？》](https://ask.csdn.net/questions/1021515?sort=votes_count)查看。**

​		**本次实例代码过多过长，实例部分并不建议观看，可直接去[《如何在Fabric SDK Go中添加集合配置？》](https://ask.csdn.net/questions/1021515?sort=votes_count)查看。**

### 配置文件

#### org1sdk-config.yaml

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
      keyfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.key
      certfile: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.cert
      
channels:
  mychannel:
    orderers:
      - orderer.example.com
    #      - orderer2.example.com
    #      - orderer3.example.com
    #      - orderer4.example.com
    #      - orderer5.example.com
    peers:
      peer0.org1.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      # Add other peers in mychannel for byfn
#      peer1.org1.example.com:
#        endorsingPeer: true
#        chaincodeQuery: true
#        ledgerQuery: true
#        eventSource: true

      peer0.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

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

#### org2sdk-config.yaml

``` yaml
version: 1.0.0
client:
  organization: Org1
  logging:
    level: info
  cryptoconfig:
    path: /home/hu/GOPATH/src/github.com/hyperledger/fabric-samples/first-network/crypto-config

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
    #      - orderer2.example.com
    #      - orderer3.example.com
    #      - orderer4.example.com
    #      - orderer5.example.com

    peers:
      peer0.org1.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      # Add other peers in mychannel for byfn
#      peer1.org1.example.com:
#        endorsingPeer: true
#        chaincodeQuery: true
#        ledgerQuery: true
#        eventSource: true

      peer0.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

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

### 依赖

go.mod

``` 
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

### 主程序

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
	chaincodePath = "github.com/hyperledger/fabric-samples/chaincode/work"
	chaincodeName = "work"
	channelName = "mychannel"
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

	InstallAndInvoke(org1Client,org2Client,"v9")
	InstallAndUpgrade(org1Client,org2Client,"v10")
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
	if err := client1.QueryChaincode(peer0Org2,"2"); err != nil {
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

	if err := client1.QueryChaincode(peer0Org1,"2"); err != nil {
		log.Panicf("链码查询失败:%v",err)
	}
	log.Println("成功在peer0.org1上使用链码查询")

}

func (client *Client) QueryChaincode( peer string, keys string) error {
	//构建查询请求
	req := channel.Request{
		ChaincodeID: client.ChaincodeID,
		Fcn: "getProductStorage",
		Args: packArgs([]string{keys}),
	}
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
	args := packArgs([]string{"2","huawei","10"})
	req := channel.Request{
		ChaincodeID: client.ChaincodeID,
		Fcn: "createProduct",
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
	//生成Collection Config
	collection1,err := createCollectionConfig("collectionProduct","OR('Org1MSP.member','Org2MSP.member')",0,3,1000000)
	if err != nil {
		log.Panicf("创建collectionconfig失败：%v\n",err)
	}
	collection2,err := createCollectionConfig("collectionProductStorage","OR('Org1MSP.member')",0,3,1000000)
	if err != nil {
		log.Panicf("创建collectionconfig失败：%v\n",err)
	}
	collectionConfigs := []*common.CollectionConfig{collection1,collection2}
	//背书策略
	org1OrOrg2 := "OR('Org1MSP.member','Org2MSP.member')"
	chaincodePolicy,err := client.genPolicy(org1OrOrg2)
	if err != nil {
		return "",errors.WithMessage(err,"gen policy from string error")
	}
	args := packArgs([]string{"init"})
	req := resmgmt.InstantiateCCRequest{
		Name: client.ChaincodeID,
		Path: client.ChaincodePath,
		Version: version,
		Args: args,
		Policy: chaincodePolicy,
		CollConfig: collectionConfigs,
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
			log.Panicf("链码 %s 已安装在节点：%s.\n",client.ChaincodeID,resp.Target)
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
	args := packArgs([]string{"init"})
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

		ChaincodeID: chaincodeName,
		ChaincodePath: chaincodePath,
		GoPath: os.Getenv("GOPATH"),
		ChannelID: channelName,
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

func createCollectionConfig(name,policy string,requiredPeerCount,maxPeerCount int32,blockToLive uint64)(*common.CollectionConfig, error){
	collectionConfig, err := newCollectionConfig(name,policy, requiredPeerCount,maxPeerCount,blockToLive)
	if err != nil {
		return nil,errors.WithMessage(err, "failed to create collection config 1")
	}
	return collectionConfig,nil
}

func newCollectionConfig(colName, policy string, reqPeerCount, maxPeerCount int32, blockToLive uint64) (*common.CollectionConfig, error) {
	p, err := cauthdsl.FromString(policy)
	if err != nil {
		return nil, err
	}
	cpc := &common.CollectionPolicyConfig{
		Payload: &common.CollectionPolicyConfig_SignaturePolicy{
			SignaturePolicy: p,
		},
	}
	return &common.CollectionConfig{
		Payload: &common.CollectionConfig_StaticCollectionConfig{
			StaticCollectionConfig: &common.StaticCollectionConfig{
				Name:              colName,
				MemberOrgsPolicy:  cpc,
				RequiredPeerCount: reqPeerCount,
				MaximumPeerCount:  maxPeerCount,
				BlockToLive:       blockToLive,
			},
		},
	}, nil
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

### 链码文件

​		这个链码文件是之前学习Fabric链码时的例子，主要内容是私密数据。上面程序所操作的链码就是本链码文件。

main.go

``` go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
	"strconv"
)

type MyChaincode struct {

}

func (MyChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	return shim.Success([]byte("链码初始化完成............."))
}

func (MyChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	//获取调用的方法及其参数
	functionName,args := stub.GetFunctionAndParameters()
	switch functionName {
	case "Test":
		fmt.Println("调用成功。。。。。。。。。。。。。。。。。。。。。")
		return shim.Success([]byte("链码调用成功。。。。。。。。。。。。。。。。。。。。。。。。"))
	case "createProduct":
		return createProduct(stub,args)
	case "getProduct":
		return getProduct(stub,args)
	case "getProductStorage":
		return getProductStorage(stub,args)
	}
	return shim.Success([]byte("链码调用失败。。。。。。。。。。。。。。。。。。。。。。。。"))
}

func getProductStorage(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error("非法参数！id string")
	}
	id := args[0]
	result,errOfGetPriData := stub.GetPrivateData("collectionProductStorage",id)
	if errOfGetPriData != nil {
		fmt.Println(errOfGetPriData)
		return shim.Error(errOfGetPriData.Error())
	}
	var productStorage ProductStorage
	errOfUnmarshal := json.Unmarshal(result,&productStorage)
	if errOfUnmarshal != nil {
		fmt.Println(errOfUnmarshal)
		return shim.Error(errOfUnmarshal.Error())
	}
	fmt.Println(productStorage)
	return shim.Success(result)
}

func getProduct(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error("非法参数！id string")
	}
	id := args[0]
	result,errOfGetPriData := stub.GetPrivateData("collectionProduct",id)
	if errOfGetPriData != nil {
		fmt.Println(errOfGetPriData)
		return shim.Error(errOfGetPriData.Error())
	}
	var product Product
	errOfUnmarshal := json.Unmarshal(result,&product)
	if errOfUnmarshal != nil {
		fmt.Println(errOfUnmarshal)
		return shim.Error(errOfUnmarshal.Error())
	}
	fmt.Println(product)
	return shim.Success(result)
}

func createProduct(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 3 {
		return shim.Error("非法参数！id string,name string,storage int")
	}
	id := args[0]
	name := args[1]
	storage,_ := strconv.Atoi(args[2])
	var product = Product{ID:id,Name:name}
	var productStorage = ProductStorage{ID:id,Storage: storage}
	data1,errOfMarshalPro := json.Marshal(product)
	if errOfMarshalPro != nil {
		fmt.Println(errOfMarshalPro)
		return shim.Error(errOfMarshalPro.Error())
	}
	data2,errOfMarshalSto := json.Marshal(productStorage)
	if errOfMarshalSto != nil {
		fmt.Println(errOfMarshalSto)
		return shim.Error(errOfMarshalSto.Error())
	}

	errOfPutPro := stub.PutPrivateData("collectionProduct",id,data1)
	if errOfPutPro != nil {
		fmt.Println(errOfPutPro)
		return shim.Error(errOfPutPro.Error())
	}
	errOfPutSto := stub.PutPrivateData("collectionProductStorage",id,data2)
	if errOfPutSto != nil {
		fmt.Println(errOfPutSto)
		return shim.Error(errOfPutSto.Error())
	}
	return shim.Success([]byte(id))
}

type Product struct {
	ID string `json:"id"`
	Name string `json:"name"`
}

type ProductStorage struct {
	ID string `json:"id"`
	Storage int `json:"storage"`
}

func main()  {
	err := shim.Start(new(MyChaincode))
	if err != nil {
		fmt.Println("链码启动失败",err)
	}
}
```





