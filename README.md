# 목표

---

- 조직 6개의 각각의 피어는 한개씩
- Orderer는 총 3개로 구성
- 채널 및 체인코드는 이후에 구성할 예정

# 이걸 할 수 있는 사람

---

- 도커 및 도커 컴포즈를 이전에 많이 다뤄본 사람
- 최소한의 리눅스를 다뤄본 사람
- 마운트의 개념을 아는 사람
- 하이퍼 레저 패브릭에 대해 개념을 숙지한 사람

# 환경 생성 순서

---

### 시작

- Hyperledger Fabric Github에서 ‘시작하기’에 다운로드 파일들을 전부 제공하기 때문에 따라하면 됨!

### 1. crpyto-config.yaml 생성

- 몇개의 orderer서버를 구성할 것인지, peer, org설정
- 이것을 해야 인증서를 이에 맞게 불러올 수 있음.
- 인증서 없으면 아무것도 못함.

```c
# OrdererOrgs: 오더러 조직 구성 정보
# - Name: 오더러 조직의 이름
#   Domain: 오더러 조직의 도메인
#   Specs: 오더러 노드의 호스트명 정보
#     - Hostname: 오더러 노드의 호스트명 (여기서는 오더러가 2개이므로 "orderer"로 고정)
#   Template: 오더러 노드의 갯수를 지정하는 정보
#     Count: 오더러 노드의 총 개수 (여기서는 2개로 지정)

OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
    Template:
      Count: 2

# PeerOrgs: 피어 조직 구성 정보
# - Name: 피어 조직의 이름
#   Domain: 피어 조직의 도메인
#   EnableNodeOUs: 피어 조직에 Node Organizational Units를 사용할지 여부 (true 또는 false)
#   Template: 피어 노드의 갯수를 지정하는 정보
#     Count: 피어 노드의 총 개수 (여기서는 각 피어 조직마다 1개로 지정)
#   Users: 피어 노드에 생성될 사용자 계정의 개수를 지정하는 정보
#     Count: 피어 노드에 생성될 사용자 계정의 총 개수 (여기서는 각 피어 조직마다 1개로 지정)

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
```

### 인증서 생성

- 설치
    
    ```c
    # Fabric Samples GitHub에서 cryptogen 이미지 가져오기
    docker pull hyperledger/fabric-tools:latest
    ```
    
- 인증서 발급
    
    ```c
    # 컨테이너를 실행하여 crypto-config.yaml 파일을 마운트하고 인증서 생성
    docker run --rm -v /Users/mac_nkm/docker_file/fabric:/opt -w /opt hyperledger/fabric-tools:latest cryptogen generate --config=crypto-config.yaml
    ```
    

```dart
cryptogen generate --config=crypto-config.yaml --output=./crypto-config
```

### crypto-config

- 인증서 생성 이후에 생기는 폴더
- 위에 작성한 peer, orderer의 맞춰서 생성해줌.
- 대박임.
- 이걸 했으면 다음 단계로 넘어가야함.

### configtx.yaml 작성

- 위에 작성한 친구의 네트워크 구성을 설정해주는 친구

```c
# Organizations: 피어 및 오더러 조직 정보를 정의하는 섹션
# 각 피어 및 오더러 조직은 고유한 이름, MSP(IDentity, Membership Service Provider) 디렉토리 경로 및 권한 정책을 가집니다.

Organizations:
    - &OrdererOrg
        Name: Orderer
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/example.com/msp
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

    - &Org1
        Name: Org1
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp
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

    # 나머지 피어 조직 (Org2, Org3, Org4, Org5)에 대해서도 동일하게 정의

# Capabilities: 채널 및 오더러 기능에 대한 기능(버전)을 정의하는 섹션
# 여기서는 V2_0 채널 및 오더러 기능을 사용 가능하도록 설정합니다.

Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true

    Orderer: &OrdererCapabilities
        V2_0: true

# Application 및 OrdererDefaults 섹션은 오더러 및 애플리케이션 기본 설정을 정의합니다.
# 기본적으로 이 부분은 애플리케이션과 오더러에 대한 구성 프로필을 설정합니다.
# 현재는 애플리케이션과 오더러에 대한 설정만 정의되어 있고, 각 조직에 대한 설정은 이후에 추가될 수 있습니다.

Application: &ApplicationDefaults
    Organizations:

Orderer: &OrdererDefaults
    OrdererType: solo
    Addresses:
        - orderer.example.com:7050
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
    Kafka:
        Brokers:
            - 127.0.0.1:9092
    Organizations:

# Profiles: 네트워크 구성 프로필을 정의합니다.
# FiveOrgsOrdererGenesis 프로필은 오더러의 제네시스 블록을 구성하는 데 사용됩니다.
# FiveOrgsChannel 프로필은 채널 구성을 정의하는 데 사용됩니다.

Profiles:
    FiveOrgsOrdererGenesis:
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *OrdererOrg
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
                - *Org3
                - *Org4
                - *Org5

    FiveOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
                - *Org3
                - *Org4
                - *Org5
        Capabilities:
            <<: *ChannelCapabilities
```

### docker-compose.yml

```dart
version: '2'

networks:
  basic:
    # 'basic' 네트워크는 컨테이너들 사이의 통신을 위한 Docker 네트워크를 정의합니다.

services:
  orderer.example.com:
    # Orderer 서비스를 정의합니다.
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      # Orderer의 환경 변수 설정
      - ORDERER_GENERAL_LOGLEVEL=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto-config/orderer/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    volumes:
      # 호스트와 컨테이너 간 디렉토리 공유를 설정합니다.
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/etc/hyperledger/crypto-config/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls:/etc/hyperledger/crypto-config/orderer/tls
      - ./configtx/genesis.block:/etc/hyperledger/configtx/genesis.block

  orderer0.example.com:
    # orderer0의 설정들
    # 나머지 Orderer 서비스들(orderer1.example.com 등)도 동일한 방식으로 정의됩니다.
    container_name: orderer0.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      - ORDERER_GENERAL_LOGLEVEL=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/crypto-config/orderer/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 8050:7050
    volumes:
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp:/etc/hyperledger/crypto-config/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls:/etc/hyperledger/crypto-config/orderer/tls
      - ./configtx/genesis.block:/etc/hyperledger/configtx/genesis.block

  peer0.org1.example.com:
    # Peer0 Org1의 설정들
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=net_basic
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
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/:/etc/hyperledger/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com
    networks:
      - basic

  # 나머지 Peer들(peer0.org2.example.com, peer0.org3.example.com 등)도 동일한 방식으로 정의됩니다.

  cli:
    # CLI 서비스의 설정들
    container_name: cli
    image: hyperledger/fabric-tools:latest
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=net_basic
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
    networks:
      - basic
```

### 기타 이후 필요한 것들

**중요! 이전에 genesis.block가 없으면 orderer가 실행이 안될 수도 있습니다. 그래서 블록을 꼭 설정해줘야 합니다.**

1. 채널 생성:
채널을 생성하려면 채널 구성 파일(**`configtx.yaml`**)을 작성해야 합니다. **`configtxgen`** 도구를 사용하여 채널의 Genesis 블록을 생성할 수 있습니다. 먼저, **`configtx.yaml`** 파일을 작성한 후에 다음 명령을 실행하여 Genesis 블록을 생성합니다:
    
    ```bash
    $ configtxgen -profile <프로파일 이름> -outputBlock ./configtx/genesis.block
    ```
    
    **`<프로파일 이름>`**은 **`configtx.yaml`** 파일에 정의된 프로파일의 이름입니다. 이 명령을 실행하면 **`genesis.block`** 파일이 생성됩니다.
    
2. 채널 생성 및 조인:
채널을 생성한 후, 이를 사용하여 피어를 채널에 조인시켜야 합니다. **`peer channel create`** 명령을 사용하여 채널을 생성하고, **`peer channel join`** 명령을 사용하여 피어를 채널에 조인시킵니다. 예를 들어, 다음과 같이 채널을 생성하고 피어를 조인시킬 수 있습니다:
    
    ```bash
    $ peer channel create -o orderer.example.com:7050 -c mychannel -f ./path/to/channel.tx --tls --cafile /path/to/orderer/tls/ca.crt
    $ peer channel join -b mychannel.block
    ```
    
    **`channel.tx`** 파일은 채널 구성 파일로, **`mychannel`** 채널을 생성하기 위해 사용합니다.
    
3. 체인코드 설치 및 인스턴스화:
체인코드를 설치하려면 체인코드 패키지를 생성한 후 피어에 설치해야 합니다. 그리고나서 채널에 체인코드를 인스턴스화시켜야 합니다. 예를 들어, 다음과 같이 체인코드를 설치하고 인스턴스화시킬 수 있습니다:
    
    ```less
    $ peer chaincode package -n mycc -v 1.0 -p github.com/mychaincode -s -S -i "OR ('Org1MSP.peer', 'Org2MSP.peer')"
    $ peer chaincode install mycc.1.0
    $ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /path/to/orderer/tls/ca.crt -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR ('Org1MSP.peer', 'Org2MSP.peer')"
    ```
    
    위 명령어는 **`mycc`**라는 이름의 체인코드를 설치하고 **`mychannel`** 채널에 인스턴스화합니다.
    
    ### 내 디렉토리 구조
    

![스크린샷 2023-07-20 오후 5.46.23.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fca5fa8b-1d6d-4989-a5bd-b5955ed10ca1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-20_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.46.23.png)

![스크린샷 2023-07-20 오후 5.47.01.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c51f8534-a35a-4896-a1e2-77dfa60d994d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-20_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.47.01.png)
