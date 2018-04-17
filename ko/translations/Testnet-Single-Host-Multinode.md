이번 튜토리얼에서는 단일 호스트상에서 멀티 노드 블록체인을 구동(_**single host, multi-node testnet**_)할 수 있도록 설정하는 방법을 설명하겠습니다. 로컬 PC에서 두 노드를 설정하고, 서로간에 통신해보도록 할 것입니다. 이 섹션에서 설명하는 예제는 세 커맨드 라인 애플리케이션을 사용합니다: `nodeos`, `keosd`, `cleos`. 구성된 테스트넷을 그림으로 그리면 아래의 다이어그램과 같게 됩니다.
![Single Host, Multi-Node Testnet](assets/Single-Host-Multi-Node-Testnet.png)

`keosd`, `cleos`, `nodeos`를 설치하고 경로설정까지 완료돼 있거나, 파일시스템 상에서 저 세 애플리케이션의 실행환경을 알고 있어야 합니다. ([로컬 환경 설정하기](https://github.com/eoseoul/docs/blob/master/ko/translations/Local-Environment.md)를 참고하세요)

그럼 이제 터미널 윈도우를 열고, 다음 순서대로 수행해 봅시다.

### 지갑 관리자(월렛 매니저) 시작
먼저, 지갑 관리 애플리케이션인 `keosd`를 실행합니다.
```
keosd --http-server-address 127.0.0.1:8899
```

잘 실행됐다면 결과로 다음과 같은 정보들이 표시됩니다:
```
2493323ms thread-0   wallet_plugin.cpp:39          plugin_initialize    ] initializing wallet plugin
2493323ms thread-0   http_plugin.cpp:141           plugin_initialize    ] host: 127.0.0.1 port: 8899
2493323ms thread-0   http_plugin.cpp:144           plugin_initialize    ] configured http to listen on 127.0.0.1:8899
2493323ms thread-0   http_plugin.cpp:213           plugin_startup       ] start listening for http requests
2493324ms thread-0   wallet_api_plugin.cpp:70      plugin_startup       ] starting wallet_api_plugin
```

표시 정보 중 지갑(wallet)이 127.0.0.1:8899를 리스닝하고 있다는 데 주목하세요. `keosd`가 제대로 수행되어 정상적으로 포트를 리스닝하고 있다는 뜻입니다. 저 라인이 표시되지 않는다면, 혹은 "starting wallet_api_plugin" 라인 앞에 오류 보고가 떠 있다면, 이슈를 진단하고 해결해야 합니다.

`keosd`룰 제대로 실행시켰다면, 지갑 애플리케이션이 계속 실행되도록 현재 윈도우는 놔 두고, 새 터미널 윈도우를 엽니다.

### 기본 지갑 생성
다음 터미널 윈도우에서 커맨트 라인 유틸리티인 `cleos`를 실행시켜서 기본 지갑을 생성합니다.
```
cleos --wallet-port 8899  wallet create
```

`cleos`는 "기본" 지갑을 생성했는지 표시해 줄 것이며, 이후에 지갑에 액세스하기 위한 패스워드를 알려줄 것입니다. 메시지에서도 안내해 주고 있듯이, 이후에 사용할 수 있도록 패스워드를 잘 보관해 둡시다. 예제에서 실행한 결과는 다음과 같습니다:
```
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JsmfYz2wrdUEotTzBamUCAunAA8TeRZGT57Ce6PkvM12tre8Sm"
```

`keosd` 쪽 윈도우에도 현재 상태 변화가 출력될 겁니다. 우리는 계속해서 두 번째 터미널의 `cleos` 명령어 이후 작업을 진행합니다.

### 첫번째 프로듀서 노드 시작
첫번째 프로듀서 노드를 시작하겠습니다. 세 번째 터미널 윈도우에서 다음과 같이 실행합니다:
```
nodeos --enable-stale-production --producer-name eosio --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin
```

이 명령어는 "bios" 프로듀서로 알려져 있는 특별한 프로듀서를 생성하게 됩니다. 지금까지 모두 정상적으로 진행됐다면, `nodeos` 프로세스의 실행 결과로 블록이 생산되었다고 출력됩니다.

### 두 번째 프로듀서 노드 시작
다음 명령어는 이 튜토리얼을 `${EOSIO_SOURCE}` 디렉토리에서 실행하고 있다고 가정한 상태에서 실행됩니다. 이 디렉토리는 빌드를 위해 `./eosio_build.sh`를 돌렸던 디렉토리입니다. 궁금한 점이 있다면 [코드 받기](https://github.com/eoseoul/docs/blob/master/ko/translations/Local-Environment.md#%EC%BD%94%EB%93%9C-%EB%B0%9B%EA%B8%B0)를 찾아보시면 자세한 정보를 확인하실 수 있습니다.

노드를 추가하기 위해서는 우선 `eosio.bios` 컨트랙트를 로드해야 합니다. 이 컨트랙트는 여러분에게 다른 계정에 대한 자원 할당과 다른 권한이 필요한 API 호출에 대한 제어권을 부여합니다. 두 번째 터미널 윈도우에서 다음 명령어를 실행해 컨트랙트를 로드하세요:
```
cleos --wallet-port 8899 set contract eosio build/contracts/eosio.bios
```

`inita` 란 계정명을 사용해 프로듀서가 될 계정을 생성할 것입니다. 계정 생성을 위해서는 계정과 연결된 키를 생성해야만 하고, 이 키를 지갑에 임포트해야 합니다.

다음과 같이 키 생성 명령어를 실행합니다:
```
cleos create key
```
새로 생성된 공개키와 개인키 쌍이 출력됩니다. 다음과 유사한 형식일 것입니다.

**중요: 이후의 커맨드 라인 명령어는 여기서 출력되는 키를 사용합니다. 이 튜토리얼에서 커맨드를 복붙하려면, 여러분이 방금 생성한 키를 사용해서는 안됩니다. 만약, 여러분이 새로 생성한 키를 사용하고 싶다면, 커맨드라인에서 키값을 여러분이 생성한 키로 변경하셔야 합니다.**

```
Private key: 5JgbL2ZnoEAhTudReWH1RnMuQS6DBeLZt4ucV6t8aymVEuYg7sr
Public key: EOS6hMjoWRF2L8x9YpeqtUEcsDKAyxSuM1APicxgRU1E3oyV5sDEg
```

개인 키 부분을 지갑에 임포트하세요. 성공했다면, 쌍으로 맺어진 공개 키가 표시될 것입니다. 위에서 생성했던 공개 키와 일치해야만 합니다:

```
cleos --wallet-port 8899 wallet import 5JgbL2ZnoEAhTudReWH1RnMuQS6DBeLZt4ucV6t8aymVEuYg7sr
imported private key for: EOS6hMjoWRF2L8x9YpeqtUEcsDKAyxSuM1APicxgRU1E3oyV5sDEg
```

이제 `inita` 계정을 생성해서 프로듀서로 만들 겁니다. `create account` 명령어에는 공개 키 두 개를 인자로 줘야 합니다. 하나는 계정의 소유자(owner) 권한용 키고, 또 하나는 계정의 활동(active) 권한용 키입니다. 이 예제에서는, 새로 생성한 공개 키를 두 번 썼습니다. 소유자랑 활동 권한을 같은 키로 설정한 것이죠. create 명령어의 실행 예제는 아래와 같습니다:

```
cleos --wallet-port 8899 create account eosio inita EOS6hMjoWRF2L8x9YpeqtUEcsDKAyxSuM1APicxgRU1E3oyV5sDEg EOS6hMjoWRF2L8x9YpeqtUEcsDKAyxSuM1APicxgRU1E3oyV5sDEg
executed transaction: d1ea511977803d2d88f46deb554f5b6cce355b9cc3174bec0da45fc16fe9d5f3  352 bytes  102400 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"inita","owner":{"threshold":1,"keys":[{"key":"EOS6hMjoWRF2L8x9YpeqtUEcsDK...
```
이제 컨트랙트를 할당받을 수 있게 된 계정을 손에 넣었습니다. 이것으로 여러가지를 할 수 있다는 뜻입니다. 다른 튜토리얼에서는 간단한 컨트랙트 설정을 위해 계정을 사용했습니다. 이번에는 계정을 블록 프로듀서로 사용해 보죠.

네 번째 터미널 윈도우를 열고, 두 번째 `nodeos` 인스턴스를 실행합니다. 첫 번째 블록 프로듀서를 생성할 때 사용했던 것 보다 명령어가 좀 더 긴 것에 주목하세요. 이건 첫 번째 `nodeos` 인스턴스와 충돌을 방지하기 위해 불가피한 일입니다. 다행히도, 아래의 명령어를 복붙하시면 됩니다(필요하다면 키 부분을 바꾸세요):

```
nodeos --producer-name inita --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin --http-server-address 127.0.0.1:8889 --p2p-listen-endpoint 127.0.0.1:9877 --p2p-peer-address 127.0.0.1:9876 --config-dir node2 --data-dir node2 --private-key [\"EOS6hMjoWRF2L8x9YpeqtUEcsDKAyxSuM1APicxgRU1E3oyV5sDEg\",\"5JgbL2ZnoEAhTudReWH1RnMuQS6DBeLZt4ucV6t8aymVEuYg7sr\"]

```

새 노드 생성 결과로 약간의 활동 정보가 출력됩니다만, 이 튜토리얼의 마지막 단계에서 `inita` 계정을 프로듀서 계정으로 만들고 활성화하기 전까지는 더이상 아무 정보도 출력되지 않을 것입니다. 새로 시작된 노드에서 출력하는 결과의 예제가 아래 표시돼 있습니다. 여러분이 실제 해 보시면 아마 약간 다를 것인데요, 이는 이 명령어를 실행하는 동안 얼마나 시간이 지났느냐에 따릅니다. 사족을 좀 더 달면, 예제로 보여드리는 것은 출력의 마지막 일부입니다:

```
2393147ms thread-0   producer_plugin.cpp:176       plugin_startup       ] producer plugin:  plugin_startup() end
2393157ms thread-0   net_plugin.cpp:1271           start_sync           ] Catching up with chain, our last req is 0, theirs is 8249 peer dhcp15.ociweb.com:9876 - 295f5fd
2393158ms thread-0   chain_controller.cpp:1402     validate_block_heade ] head_block_time 2018-03-01T12:00:00.000, next_block 2018-04-05T22:31:08.500, block_interval 500
2393158ms thread-0   chain_controller.cpp:1404     validate_block_heade ] Did not produce block within block_interval 500ms, took 3061868500ms)
2393512ms thread-0   producer_plugin.cpp:241       block_production_loo ] Not producing block because production is disabled until we receive a recent block (see: --enable-stale-production)
2395680ms thread-0   net_plugin.cpp:1385           recv_notice          ] sync_manager got last irreversible block notice
2395680ms thread-0   net_plugin.cpp:1271           start_sync           ] Catching up with chain, our last req is 8248, theirs is 8255 peer dhcp15.ociweb.com:9876 - 295f5fd
2396002ms thread-0   producer_plugin.cpp:226       block_production_loo ] Previous result occurred 5 times
2396002ms thread-0   producer_plugin.cpp:244       block_production_loo ] Not producing block because it isn't my turn, its eosio

```

이 시점에서 두 번째 `nodeos`는 놀고 있는 프로듀서입니다. 활동하는(블록을 생산하는) 프로듀서로 바꾸려면 `inita`를 bios 노드에 프로듀서로 등록해야만 합니다. 아울러 bios 노드에서는 프로듀서 스케줄을 업데이트할 액션을 실행해야 합니다.

```
cleos --wallet-port 8899 push action eosio setprods "{ \"version\": 1, \"producers\": [{\"producer_name\": \"inita\",\"block_signing_key\": \"EOS6hMjoWRF2L8x9YpeqtUEcsDKAyxSuM1APicxgRU1E3oyV5sDEg\"}]}" -p eosio@active
executed transaction: 2cff4d96814752aefaf9908a7650e867dab74af02253ae7d34672abb9c58235a  272 bytes  105472 cycles
#         eosio <= eosio::setprods              {"version":1,"producers":[{"producer_name":"inita","block_signing_key":"EOS6hMjoWRF2L8x9YpeqtUEcsDKA...
```

축하합니다, 방금 두 노드로 이루어진 테스트넷을 설정하는 데 성공했습니다! 원래 노드(bios 노드)는 이제 블록을 생산하지는 않고 수신만 하는 것을 확인해 보실 수 있습니다. `get info` 명령어를 각 노드에 때려보시면 확인 가능합니다.

첫 번째 노드에 대한 `get info` 결과:
```
cleos get info
```

결과는 반드시 다음과 유사해야 합니다:
```
{
  "server_version": "223565e8",
  "head_block_num": 11412,
  "last_irreversible_block_num": 11411,
  "head_block_id": "00002c94daf7dff456cd940bd585c4d9b38e520e356d295d3531144329c8b6c3",
  "head_block_time": "2018-04-06T00:06:14",
  "head_block_producer": "inita"
}
```

이번엔 두 번째 노드:
```
cleos --port 8889 get info
```

결과는 반드시 다음과 유사해야 합니다:
```
{
  "server_version": "223565e8",
  "head_block_num": 11438,
  "last_irreversible_block_num": 11437,
  "head_block_id": "00002cae32697444fa9a2964e4db85b5e8fd4c8b51529a0c13e38587c1bf3c6f",
  "head_block_time": "2018-04-06T00:06:27",
  "head_block_producer": "inita"
}
```

이후 튜토리얼에서는 멀티 호스트, 멀티 노드 테스트넷을 구성하기 위한 보다 발전된 도구들을 어떻게 사용하는지 알아보도록 하겠습니다.

## 번역 정보

* 위키 원문 : https://github.com/eosio/eos/wiki/Testnet-Single-Host-Multinode
* 번역 기준 리비전 : 0c18223ea118597c7b2476118091c4094be6af99
