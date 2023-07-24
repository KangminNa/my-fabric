# 하이퍼레저패브릭을 이용한 구축 방법

---

### docker-compose.yml 이용

- mac에서 Vm깔기 싫어서 docker-compose로 peer, orderer, org를 설정 한 이후에 배포
- 체인코드 및 채널을 하기 전에 구조를 먼저 잡고 진행 할 예정
- fabric-samples 사용하여 듀토리얼 진행하다가 공부 겸 직접 구축하기로 마음 먹음
- 한달 인턴 중 블록체인 공부하면서 기록

### 목표

- orderer 3개
- org(peer) 6개
- cli 1개

# 순서

### Docker-compose를 사용할 수 있게 관련 프로그램 설치!

- 각종 블로그를 보면서 따라하면 됩니다.

### crypto-config.yaml 작성

```bash
# Copyright bysssss Corp. All Rights Reserved.

# Template : 구성할 해당 node의 갯수.
# Users : 각 node의 사용자계정 갯수.

OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
    Template:
      Count: 2

    

PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: Org3
    Domain: org3.example.com
    EnableNodeOUs: true
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: Org4
    Domain: org4.example.com
    EnableNodeOUs: true
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: Org5
    Domain: org5.example.com
    EnableNodeOUs: true
    Template:
      Count: 1
    Users:
      Count: 1
```

### Fabric Samles Github에서 cryptogen 이미지 가져오기

```bash
# Fabric Samples GitHub에서 cryptogen 이미지 가져오기
docker pull hyperledger/fabric-tools:latest
```

### 인증서 발급

```bash
# 컨테이너를 실행하여 crypto-config.yaml 파일을 마운트하고 인증서 생성
docker run --rm -v <본인이 다운받은 혹은 프로젝트 실행 부분 위치>fabric:/opt -w /opt hyperledger/fabric-tools:latest cryptogen generate --config=crypto-config.yaml
```

- crypto-config.yaml 작성한 곳에서 실행

### crypto-config 폴더 생성

- 위와 같은 절차를 걸치면 인증서가 발급된 폴더가 생성
- orderer와 peer 노드에 각각 인증서들이 잔뜩 생겼을 것이다.

### configtx.yaml 작성

```bash
Organizations:
  - &OrdererOrg
    Name: Orderer
    ID: OrdererMSP
    MSPDir: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/msp
    Policies:
      Readers:
        Type: Signature
        Rule: "OR('OrdererMSP.member')"
      Writers:
        Type: Signature
        Rule: "OR('OrdererMSP.member')"
      Admins:
        Type: Signature
        Rule: "OR('OrdererMSP.admin')"
    OrdererEndpoints:
      - "orderer.example.com:7050"

  - &Org1
    Name: Org1
    ID: Org1MSP
    MSPDir: /etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
    Policies:
      Readers:
        Type: Signature
        Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
      Writers:
        Type: Signature
        Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
      Admins:
        Type: Signature
        Rule: "OR('Org1MSP.admin')"
    AnchorPeers:
      - Host: peer0.org1.example.com
        Port: 7051

  - &Org2
    Name: Org2
    ID: Org2MSP
    MSPDir: /etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
    Policies:
      Readers:
        Type: Signature
        Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
      Writers:
        Type: Signature
        Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
      Admins:
        Type: Signature
        Rule: "OR('Org2MSP.admin')"
    AnchorPeers:
      - Host: peer0.org2.example.com
        Port: 7051

  - &Org3
    Name: Org3
    ID: Org3MSP
    MSPDir: /etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
    Policies:
      Readers:
        Type: Signature
        Rule: "OR('Org3MSP.admin', 'Org3MSP.peer', 'Org3MSP.client')"
      Writers:
        Type: Signature
        Rule: "OR('Org3MSP.admin', 'Org3MSP.client')"
      Admins:
        Type: Signature
        Rule: "OR('Org3MSP.admin')"
    AnchorPeers:
      - Host: peer0.org3.example.com
        Port: 7051

  - &Org4
    Name: Org4
    ID: Org4MSP
    MSPDir: /etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
    Policies:
      Readers:
        Type: Signature
        Rule: "OR('Org4MSP.admin', 'Org4MSP.peer', 'Org4MSP.client')"
      Writers:
        Type: Signature
        Rule: "OR('Org4MSP.admin', 'Org4MSP.client')"
      Admins:
        Type: Signature
        Rule: "OR('Org4MSP.admin')"
    AnchorPeers:
      - Host: peer0.org4.example.com
        Port: 7051

  - &Org5
    Name: Org5
    ID: Org5MSP
    MSPDir: /etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
    Policies:
      Readers:
        Type: Signature
        Rule: "OR('Org5MSP.admin', 'Org5MSP.peer', 'Org5MSP.client')"
      Writers:
        Type: Signature
        Rule: "OR('Org5MSP.admin', 'Org5MSP.client')"
      Admins:
        Type: Signature
        Rule: "OR('Org5MSP.admin')"
    AnchorPeers:
      - Host: peer0.org5.example.com
        Port: 7051

Capabilities:
  Channel: &ChannelCapabilities
    V2_0: true

  Orderer: &OrdererCapabilities
    V2_0: true
  
  Application: &ApplicationCapabilities
    V2_0: true

Application: &ApplicationDefaults
    Policies: &ApplicationDefaultPolicies
      Readers:
        Type: ImplicitMeta
        Rule: "ANY Readers"
      Writers:
        Type: ImplicitMeta
        Rule: "ANY Writers"
      Admins:
        Type: ImplicitMeta
        Rule: "MAJORITY Admins"
      LifecycleEndorsement:
        Type: ImplicitMeta
        Rule: "ANY Readers"
      Endorsement:
        Type: ImplicitMeta
        Rule: "ANY Writers"
    Organizations:
    Capabilities:
        <<: *ApplicationCapabilities
             
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
    ChannelCreation:
      Type: ImplicitMeta
      Rule: "ANY Readers"
  Capabilities:
    <<: *ChannelCapabilities
        

Orderer: &OrdererDefaults
  OrdererType: etcdraft
  Addresses:
    - orderer.example.com:7050
  BatchTimeout: 1s
  BatchSize:
    MaxMessageCount: 30
    AbsoluteMaxBytes: 99MB
    PreferredMaxBytes: 512KB
  EtcdRaft:
    Consenters:
      - Host: orderer.example.com
        Port: 7050
        ClientTLSCert: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
        ServerTLSCert: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
      - Host: orderer0.example.com
        Port: 7050
        ClientTLSCert: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
        ServerTLSCert: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
      - Host: orderer1.example.com
        Port: 7050
        ClientTLSCert: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
        ServerTLSCert: /etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
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
  Capabilities:
    <<: *OrdererCapabilities

Profiles:
  ExampleGenesis:
    <<: *ChannelDefaults
    Orderer:
      <<: *OrdererDefaults
      Organizations:
        - *Org1
        - *Org2
        - *Org3
        - *Org4
        - *Org5
      Capabilities:
        <<: *OrdererCapabilities
    Application:
      <<: *ApplicationDefaults
      Organizations:
        - *Org1
        - *Org2
        - *Org3
        - *Org4
        - *Org5
    Consortiums:
      SampleConsortium:
        Organizations:
          - *Org1
          - *Org2
          - *Org3
          - *Org4
          - *Org5

  example-channel:
    Consortium: SampleConsortium
    <<: *ApplicationDefaults
    Application:
      <<: *ApplicationDefaults
      Capabilities:
        <<: *ApplicationCapabilities 
      Organizations:
        - *Org1
        - *Org2
        - *Org3
        - *Org4
        - *Org5
```

- 눈물 흘린 곳. 버전에 따라 작성 법이 조금씩 달라지고 요구 조건이 달라지기 때문에 삽질을 많이 해야함.
- Policies 작성이 필요한 곳에는 꼭 적어줘야함
- Policies에서 꼭 “ANY”, “ADMIN” 등 권한자를 꼭 대문자로 적어야 함.
- 참고로 제대로 된 이해 없이 작성한 부분이기 때문에 나중에 고쳐야 겠지만 현재는 조잡한 내 머리로 했음.
- 다 필요한 문장들이 아닐 겁니다.
- 일단 꼭 channel과 genesis를 작성해야 함
- 그래야 다음에 channel과 genesis 블록을 토대로 orderer가 작동할 거임.

### docker-compose.yml 작성

```bash
version: '2'

networks:
  basic:
    

services:
  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      - ORDERER_GENERAL_LOGLEVEL=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    networks:
      - basic

    volumes:
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls

  orderer0.example.com:
    container_name: orderer0.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      - ORDERER_GENERAL_LOGLEVEL=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 8050:7050
    networks:
      - basic

    volumes:
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls

  orderer1.example.com:
    container_name: orderer1.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      - ORDERER_GENERAL_LOGLEVEL=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 9050:7050
    networks:
      - basic

    volumes:
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=basic
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=debug
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7051:7051
    volumes:
      - /var/run/:/host/var/run/
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com
    networks:
      - basic

  peer0.org2.example.com:
    # Peer0 Org2의 설정들
    container_name: peer0.org2.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=basic
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=debug
      - CORE_PEER_ID=peer0.org2.example.com
      - CORE_PEER_ADDRESS=peer0.org2.example.com:7052
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org2.example.com:7052
      - CORE_PEER_LOCALMSPID=Org2MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7052:7052
    volumes:
      - /var/run/:/host/var/run/
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com
    networks:
      - basic

  peer0.org3.example.com:
    # Peer0 Org3의 설정들
    container_name: peer0.org3.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=basic
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=debug
      - CORE_PEER_ID=peer0.org3.example.com
      - CORE_PEER_ADDRESS=peer0.org3.example.com:7053
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org3.example.com:7053
      - CORE_PEER_LOCALMSPID=Org3MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com
    networks:
      - basic

  peer0.org4.example.com:
    # Peer0 Org4의 설정들
    container_name: peer0.org4.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=basic
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=debug
      - CORE_PEER_ID=peer0.org4.example.com
      - CORE_PEER_ADDRESS=peer0.org4.example.com:7054
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org4.example.com:7054
      - CORE_PEER_LOCALMSPID=Org4MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto-config/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7054:7054
    volumes:
      - /var/run/:/host/var/run/
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org4.example.com/peers/peer0.org4.example.com
    networks:
      - basic

  peer0.org5.example.com:
    # Peer0 Org5의 설정들
    container_name: peer0.org5.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=basic
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=debug
      - CORE_PEER_ID=peer0.org5.example.com
      - CORE_PEER_ADDRESS=peer0.org5.example.com:7055
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org5.example.com:7055
      - CORE_PEER_LOCALMSPID=Org5MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto-config/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7055:7055
    volumes:
      - /var/run/:/host/var/run/
      - ./configtx/:/etc/hyperledger/configtx
      - ./crypto-config/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org5.example.com/peers/peer0.org5.example.com
    networks:
      - basic

  cli:
    # CLI 서비스의 설정들
    container_name: cli
    image: hyperledger/fabric-tools:latest
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=basic
      - CORE_LOGGING_LEVEL=info
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
      - CORE_CHAINCODE_KEEPALIVE=10
      - CORE_CHAINCODE_LOGGING_LEVEL=info
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./configtx/:/etc/hyperledger/configtx
      - ./chaincode/:/etc/hyperledger/chaincode
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com:/etc/hyperledger/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com
    networks:
      - basic
```

- cli를 꼭 해야함. genesis.block, 채널 트랜잭션을 생성하려면 docker-compose up -d로 컨테이너를 올린 후에 orderer들의 나머지 설정들을 해줘야하기 때문.
- 마운트를 자기에 맞춰서 꼭 해주세요 꼭.
- 마운트 꼭!!!!!!!!!!! 자기한테 맞춰서 잘 하셔야 합니다!!!!!!!!!!!!!!!!!!!!!!!!!!!
- 도커 local 환경에서는 자기와 같은 네트워크 상에서만 다른 컨테이너끼리 통신이 가능합니다. 그렇기때문에 꼭 네트워크 환경도 신경써주세요.

### 만약 여기까지 왔다면

![스크린샷 2023-07-21 오후 4.56.34.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e44b1b60-9f8b-435a-ad44-a38e6aea371f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.56.34.png)

### docker-compose exec cli bash

- cli 컨테이너에 접속하여 자신에게 맞는 configtx.yaml 위치로 이동
- 이후 genesis.block와 채널 트랜잭션을 생성

### genesis.block 파일생성

```bash
configtxgen -profile ExampleGenesis -configPath ./ -channelID example-channel -outputBlock ./genesis.block
```

- 꼭 configtx.yaml 위치에서 실행해야 맘 편함
- configPath를 꼭 해야 실행이 되는데 안하면 환경변수 설정을 해야 함
- 그냥 들어가서 ./ 만 처리해주면 속편함

### 채널 트랜잭션 생성

```bash
configtxgen -profile example-channel -outputCreateChannelTx <아무이름>.tx -channelID example-channel -configPath ./
```

- 위와 마찬가지.

# 현재상황

---

### core.yaml 작성

- TLS 설정을 orderer.example.com에다가만 해놨음.
- 나머지 orderer0, 1은 작동이 안되고 있는 상태
- 간단 실습을 맞췄기 떄문에 이제 개념 공부를 다시 잡고 웹과 연동하여 블록체인 실제 서비스 연습을 해볼 예정.
- 다음 프로젝트에서 진행!!
