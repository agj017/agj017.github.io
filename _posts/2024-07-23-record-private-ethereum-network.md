---
layout: post
title: record-private-ethereum-network
date: 2024-07-23
category: blockchain
---

# 0. build

- geth: `make all`

- teku: `./gradlew distTar installDist`

# 1. bootnode

## 목적

각각의 peer들의 서로 connect하기 위해 사용되는 node ([설명](https://ethereum.org/en/developers/docs/nodes-and-clients/bootnodes/))

## 실행

```sh
$ bootnode --genkey=boot.key
$ bootnode --nodekey=boot.key -verbosity 5
```

`--genkey=boot.key`: key 생성

`--nodekey=boot.key`: 생성된 key path 입력

`-verbosity`: log level 설정

## 명령어 실행 후 console

```sh
enode://6407732ad9fd3b266b2ce1776feb12a05d461e524b1a598296f3e429703fd1ddbdcc18d4d918e2f587bc68f2af936bf74c3bfac4c0f56405ad8e3716cbac0db9@127.0.0.1:0?discport=30301
Note: you're using cmd/bootnode, a developer tool.
We recommend using a regular node as bootstrap node for production deployments.
INFO [07-23|13:07:42.494] New local node record                    seq=1,721,707,662,492 id=50875895c80d91f3 ip="invalid IP" udp=0 tcp=0
```

- enode(`enode://6407732ad9fd3b266b2ce1776feb12a05d461e524b1a598296f3e429703fd1ddbdcc18d4d918e2f587bc68f2af936bf74c3bfac4c0f56405ad8e3716cbac0db9@127.0.0.1:0?discport=30301`)은 bootnode의 identitifer로서 geth node들은 해당 id를 통해서 다른 peer들을 찾음

# 2. execute client (geth)

## 목적

EVM를 이용하여 smart contract를 실행하고 transaction을 broad casting 하는 node

## 실행

### genesis.json를 통한 초기설정

```json
{
  "config": {
    "chainId": 10501,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "berlinBlock": 0,
    "londonBlock": 0,
    "terminalTotalDifficultyPassed": true
  },
  "alloc": {
    "0x186E1b3f1C64C0b9c89c9Aeee7161B15E606E82d": {
      "balance": "111111111"
    }
  },
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x20000",
  "extraData": "",
  "gasLimit": "0x2fefd8",
  "nonce": "0x0000000000000042",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
```

- github의 geth [readme](https://github.com/ethereum/go-ethereum?tab=readme-ov-file#defining-the-private-genesis-state) 기반으로 작성

- `chainId`를 `10501`로 설정

- `0x186E1b3f1C64C0b9c89c9Aeee7161B15E606E82d` 사용자에게 잔고를 설정

- `terminalTotalDifficultyPassed`를 true 설정하지 않는 경우 아래와 같은 에러 발생 정확한 원인은 모르겠음 ([issue1](https://github.com/ethereum/go-ethereum/issues/30120), [issue2](https://github.com/ethereum/go-ethereum/issues/29251))

```sh
Fatal: Failed to register the Ethereum service: only PoS networks are supported, please transition old ones with Geth v1.13.x
```

### genesis.json 적용

```
geth --datadir ./data init genesis.json
```

- 명령어 실행 후 `data` 디렉토리가 생성 됨

- 크게 디렉토리 구조는 account 정보를 관리하는 `keystore`, block 정보을 관리하는 `geth` 나누어짐 자세한 구조는 아래 참조

```
data
├── geth
│   ├── LOCK
│   ├── blobpool
│   │   ├── limbo
│   │   │   ├── bkt_00004096.bag
│   │   │   ├── bkt_00135168.bag
│   │   │   ├── bkt_00266240.bag
│   │   │   ├── bkt_00397312.bag
│   │   │   ├── bkt_00528384.bag
│   │   │   ├── bkt_00659456.bag
│   │   │   ├── bkt_00790528.bag
│   │   │   ├── bkt_00921600.bag
│   │   │   ├── bkt_01052672.bag
│   │   │   ├── bkt_01183744.bag
│   │   │   ├── bkt_01314816.bag
│   │   │   ├── bkt_01445888.bag
│   │   │   ├── bkt_01576960.bag
│   │   │   ├── bkt_01708032.bag
│   │   │   └── bkt_01839104.bag
│   │   └── queue
│   │       ├── bkt_00004096.bag
│   │       ├── bkt_00135168.bag
│   │       ├── bkt_00266240.bag
│   │       ├── bkt_00397312.bag
│   │       ├── bkt_00528384.bag
│   │       ├── bkt_00659456.bag
│   │       ├── bkt_00790528.bag
│   │       ├── bkt_00921600.bag
│   │       ├── bkt_01052672.bag
│   │       ├── bkt_01183744.bag
│   │       ├── bkt_01314816.bag
│   │       ├── bkt_01445888.bag
│   │       ├── bkt_01576960.bag
│   │       ├── bkt_01708032.bag
│   │       └── bkt_01839104.bag
│   ├── chaindata
│   │   ├── 000004.sst
│   │   ├── 000005.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── MANIFEST-000001
│   │   ├── MANIFEST-000006
│   │   ├── OPTIONS-000007
│   │   └── ancient
│   │       ├── chain
│   │       │   ├── FLOCK
│   │       │   ├── bodies.0000.cdat
│   │       │   ├── bodies.cidx
│   │       │   ├── bodies.meta
│   │       │   ├── diffs.0000.rdat
│   │       │   ├── diffs.meta
│   │       │   ├── diffs.ridx
│   │       │   ├── hashes.0000.rdat
│   │       │   ├── hashes.meta
│   │       │   ├── hashes.ridx
│   │       │   ├── headers.0000.cdat
│   │       │   ├── headers.cidx
│   │       │   ├── headers.meta
│   │       │   ├── receipts.0000.cdat
│   │       │   ├── receipts.cidx
│   │       │   └── receipts.meta
│   │       └── state
│   │           ├── FLOCK
│   │           ├── account.data.0000.cdat
│   │           ├── account.data.cidx
│   │           ├── account.data.meta
│   │           ├── account.index.0000.cdat
│   │           ├── account.index.cidx
│   │           ├── account.index.meta
│   │           ├── history.meta.0000.rdat
│   │           ├── history.meta.meta
│   │           ├── history.meta.ridx
│   │           ├── storage.data.0000.cdat
│   │           ├── storage.data.cidx
│   │           ├── storage.data.meta
│   │           ├── storage.index.0000.cdat
│   │           ├── storage.index.cidx
│   │           └── storage.index.meta
│   ├── jwtsecret
│   ├── lightchaindata
│   │   ├── 000002.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── MANIFEST-000001
│   │   ├── OPTIONS-000003
│   │   └── ancient
│   │       ├── chain
│   │       │   ├── FLOCK
│   │       │   ├── bodies.0000.cdat
│   │       │   ├── bodies.cidx
│   │       │   ├── bodies.meta
│   │       │   ├── diffs.0000.rdat
│   │       │   ├── diffs.meta
│   │       │   ├── diffs.ridx
│   │       │   ├── hashes.0000.rdat
│   │       │   ├── hashes.meta
│   │       │   ├── hashes.ridx
│   │       │   ├── headers.0000.cdat
│   │       │   ├── headers.cidx
│   │       │   ├── headers.meta
│   │       │   ├── receipts.0000.cdat
│   │       │   ├── receipts.cidx
│   │       │   └── receipts.meta
│   │       └── state
│   │           ├── FLOCK
│   │           ├── account.data.0000.cdat
│   │           ├── account.data.cidx
│   │           ├── account.data.meta
│   │           ├── account.index.0000.cdat
│   │           ├── account.index.cidx
│   │           ├── account.index.meta
│   │           ├── history.meta.0000.rdat
│   │           ├── history.meta.meta
│   │           ├── history.meta.ridx
│   │           ├── storage.data.0000.cdat
│   │           ├── storage.data.cidx
│   │           ├── storage.data.meta
│   │           ├── storage.index.0000.cdat
│   │           ├── storage.index.cidx
│   │           └── storage.index.meta
│   ├── nodekey
│   ├── nodes
│   │   ├── 000001.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000000
│   └── transactions.rlp
└── keystore
```

### geth code 수정

[bootnodes.go](https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go) 의 `MainnetBootnodes` 배열에 bootnode enode 값 추가

### geth 실행

```sh
geth \
       --port 30304 \
       --networkid 10501 \
       --datadir ./data \
       --authrpc.addr localhost \
       --authrpc.port 8551 \
       --authrpc.vhosts localhost \
       -mine \
       --miner.etherbase=186E1b3f1C64C0b9c89c9Aeee7161B15E606E82d \
       --syncmode "snap" \
       --verbosity 5 \
       --bootnodes "enode://6407732ad9fd3b266b2ce1776feb12a05d461e524b1a598296f3e429703fd1ddbdcc18d4d918e2f587bc68f2af936bf74c3bfac4c0f56405ad8e3716cbac0db9@127.0.0.1:30301"
```

- `--port`: geth process의 port 번호

- `--networkid`: private networkd id

- `--datadir`: block 및 account 정보를 저장하는 directory path

- `--authrpc.addr`: consensus client가 접근하기 위해 사용되는 ip 정보

- `--authrpc.port`: consensus client 가 접근하기 위해 사용되는 port 정보

- `--authrpc.vhosts`: consensus client 가 접근하기 위해 사용되는 virtual hostnames 정보

- `--verbosity`: log level 설정

- `-bootnodes`: 위에서 실행한 bootnode의 enode 정보

# 3. bootnode와 geth node 통신

bootnode와 geth node가 통신했을 때의 log는 아래와 같다.

- bootnode

```sh
TRACE[07-23|14:30:01.702] << FINDNODE/v4                           id=6334139b7a7fffdf addr=127.0.0.1:30305     err=<nil>
TRACE[07-23|14:30:01.702] >> NEIGHBORS/v4                          id=6334139b7a7fffdf addr=127.0.0.1:30305     err=<nil>
TRACE[07-23|14:30:04.442] << PING/v4                               id=6334139b7a7fffdf addr=127.0.0.1:30305     err=<nil>
TRACE[07-23|14:30:04.442] >> PONG/v4                               id=6334139b7a7fffdf addr=127.0.0.1:30305     err=<nil>
TRACE[07-23|14:30:04.443] << ENRREQUEST/v4                         id=6334139b7a7fffdf addr=127.0.0.1:30305     err=<nil>
TRACE[07-23|14:30:04.443] >> ENRRESPONSE/v4                        id=6334139b7a7fffdf addr=127.0.0.1:30305     err=<nil>
```

- geth

```sh
TRACE[07-23|14:27:47.133] >> FINDNODE/v4                           id=50875895c80d91f3 addr=127.0.0.1:30301       err=<nil>
TRACE[07-23|14:27:47.134] << NEIGHBORS/v4                          id=50875895c80d91f3 addr=127.0.0.1:30301       err=<nil>
TRACE[07-23|14:27:47.177] << PING/v4                               id=50875895c80d91f3 addr=127.0.0.1:30301       err=<nil>
TRACE[07-23|14:27:47.177] >> PONG/v4                               id=50875895c80d91f3 addr=127.0.0.1:30301       err=<nil>
TRACE[07-23|14:27:47.178] << ENRREQUEST/v4                         id=50875895c80d91f3 addr=127.0.0.1:30301       err=<nil>
TRACE[07-23|14:27:47.178] >> ENRRESPONSE/v4                        id=50875895c80d91f3 addr=127.0.0.1:30301       err=<nil>
```

# 4. consensus client (teku)

## 목적

- PoS를 통한 합의 알고리즘은 consensus clinet를 통해서 실행된다.

- Ethereum2.0은 execute client + consensus client로 하나의 노드가 구성된다.

## 실행

### jwt secret 생성

```sh
openssl rand -hex 32 | tr -d "\n" > jwtsecret.hex
```

### teku 설정

yaml 파일로 설정 가능하며 설정 정보는 아래와 같음

```yaml
# network
network:
  id: 10501

ee-endpoint: http://localhost:8551
ee-jwt-secret-file: jwtsecret.hex

# database
data-path: ./tekudata
data-storage-mode: archive

# logging
log-include-validator-duties-enabled: true
```

- `id`: private network id

- `ee-endpoint`: geth 접근정보

- `ee-jwt-secret-file`: jwt path

## 명령어 실행 후 console

```sh
2024-07-23T06:02:45.176057Z main INFO Configuring logging for destination: console and file
2024-07-23T06:02:45.186567Z main INFO Logging file location: ./tekudata/logs/teku.log
2024-07-23T06:02:45.186695Z main INFO Logging includes events: true
2024-07-23T06:02:45.186760Z main INFO Logging includes validator duties: true
2024-07-23T06:02:45.186845Z main INFO Logging includes color: true
2024-07-23T06:02:45.246618Z main INFO Include P2P warnings set to: false
Unable to load configuration for network "{id=10501}": Could not load spec config from {id=10501}

To display full help:
teku [COMMAND] --help
```

- 네트워크 설정에 문제가 있는건지? private 네트워크를 지원하지 않는건지 모르겠음

## 부록

- private network 구축을 위해서 이런 [방법](https://geth.ethereum.org/docs/fundamentals/kurtosis)이 제공됨.
