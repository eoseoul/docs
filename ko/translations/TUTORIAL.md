## EOS 개발자 튜토리얼 

이 튜토리얼은 로컬 블록체인을 구성하고 스마트 컨트랙트를 테스트하는 방법을 다룹니다. 이 튜토리얼의 첫 번째 파트는 다음과 같이 구성되어 있습니다.

1. 노드 시작하기
2. 지갑 만들기
3. 계정 만들기
4. 컨트랙트를 블록체인에 올리기
5. 컨트랙트를 사용하기

두 번째 파트에서는 스마트 컨트랙트를 만들고 블록체인에 배포하는 방법을 설명합니다.

이 튜토리얼을 진행하기 위해서는 EOSIO 소프트웨어가 설치되어 있어야 하고 `nodeos`와 `cleos`가 PATH 환경변수에 등록된 디렉토리에 설치되어 있어야 합니다.

(역자 주: `eosio`를 빌드하면 build/programs 디렉토리 아래에 `nodeos`와 `cleos` 디렉토리 아래 실행 바이너리가 있습니다. 상대 경로를 사용해도 무방합니다.)


## 프라이빗 블록체인 시작하기

다음 한 줄의 명령어로 하나의 node로만 실행되는 프라이빗 블록체인을 만들 수 있습니다.

```
$ nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin 
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs
```

명령어 가장 앞쪽의 `nodeos`는 node를 실행합니다.
이 명령어는 튜토리얼 진행에 필요한 두 가지 플래그를 사용하고, 세 가지 플러그인을 로딩합니다. 위 명령어가 문제없이 실행이 되었다면 0.5초마다 블록이 생성되는 로그 메시지를 확인할 수 있습니다.

```
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
```

이 로그는 다음 세 가지 정보를 알려줍니다. 1. 방금 만든 블록체인이 라이브 상태이다. 2. 블록이 잘 생성되고 있다. 3. 블록체인을 사용할 수 있는 상태이다.

`nodeos` 명령어의 더 많은 정보를 얻으려면 다음 명령어를 사용하세요.

```
nodeos --help
```

## 지갑 만들기

지갑은 블록체인에서 액션을 승인할 때 필요한 비밀키들의 저장소입니다. 이 키들은 사용자를 위해 생성된 비밀번호를 이용해 암호화된 상태로 디스크에 저장됩니다. 
이 비밀키는 반드시 안전한 패스워드 관리 도구에 저장되어 있어야 합니다.

```
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub"
```

간단한 개발 환경의 용도로, 여러분의 지갑은 `nodeos`를 시작할 때 활성화했던 `eosio::wallet_api_plugin`을 통해 로컬 노드에 의해 관리되고 있습니다. `nodeos`을 실행할 때마다, 지갑 안의 키들을 사용하기 위해선 "잠금 해제" 과정을 거쳐야 합니다.

```
$ cleos wallet unlock --password PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub
Unlocked: default
```

이렇게 지갑을 잠금 해제했습니다. 다만 비밀번호가 화면에 노출되죠? 실제로 명령어에 바로 비밀번호를 사용하면 bash 쉘 로그에 남기 때문에 보안상 권장되는 방식은 아닙니다. 따라서 보통 다음과 같은 방식을 사용합니다.

```
$ cleos wallet unlock
password:
```

이렇게 하면 비밀번호를 interactive mode에서 입력받을 수 있습니다. 이때, 사용자는 비밀번호를 입력할 수 있고 입력한 비밀번호는 화면에 노출되지 않습니다. 지갑을 사용하지 않으실 때는 지갑을 다시 잠그는 것이 보안에 좋습니다. `nodeos`를 종료하지 않으면서 지갑을 잠그려면 다음 명령어를 실행하면 됩니다.

```
$ cleos wallet lock
Locked: default
```

지갑의 잠금과 잠금 해제에 대해서 설명했습니다. 남은 튜토리얼을 진행하려면 지갑을 잠금 해제한 상태로 두면 됩니다.

모든 블록체인은 `eosio`이라는 유일한 초기화 계정의 master key로 시작합니다. 블록체인을 이용하기 위해서는 `eosio` 계정의 비밀키를 지갑에 담아야 합니다.

다음 명령어를 통해 `eosio`의 master key를 지갑에 담겠습니다. master key는 `nodeos` 설정 디렉토리의 `config.ini` 파일에 있습니다. 튜토리얼에서는 기본 설정 디렉토리를 사용했습니다. 리눅스 시스템의 경로는 `~/.local/share/eosio/nodeos/config`이고, MacOS 시스템의 경로는 `~/Library/Application Support/eosio/nodeos/config`입니다.


```
$ cleos wallet import 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

## BIOS 컨트랙트 불러오기

이제 지갑에 `eosio` 계정의 비밀키를 담았기 때문에 초기 시스템 컨트랙트를 설정할 수 있습니다. 개발 용도로 디폴트 `eosio.bios` 컨트랙트를 사용할 수 있습니다. 이 `eosio.bios` 컨트랙트를 이용하여 다른 계정의 자원 할당을 직접 제어하고 다른 권한 관련 api를 호출할 수 있습니다. 퍼블릭 블록체인에서는 `eosio.bios` 컨트랙트를 통해 일반 컨트랙트의 메모리, 네트워크 및 CPU 활동을 위한 대역폭을 예약할 수 있는 토큰의 staking과 unstaking을 관리할 수 있습니다.

`eosio.bios` 컨트랙트는 EOSIO 소스 코드의 `contracts/eosio.bios` 디렉토리에 있습니다. `eosio` 소스의 루트 경로에서 `eosio.bios`를 실행시킬 때 아래의 명령어를 사용할 수 있습니다. 만약 다른 위치에서 `eosio.bios`를 실행하고 싶다면 `eosio.bios`의 절대 경로를 입력하면 됩니다.

```
$ cleos set contract eosio build/contracts/eosio.bios -p eosio
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

`cleos`는 두 가지 액션(`eosio::setcode`, `eosio::setabi`)을 실행하는 트랜잭션 하나를 만듭니다.

코드는 컨트랙트 실행 로직을 정의하고, ABI는 인수를 JSON과 바이너리 사이의 변환 방식을 정의합니다. 기술적으로 보자면 ABI 정의는 꼭 해야 하는 작업은 아니지만, EOSIO 유틸리티는 사용 편의를 위해서 ABI 정의가 필요합니다.

트랜잭션을 실행할 때마다 다음과 같은 형태의 출력을 볼 수 있습니다.

```
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

풀이하면 이렇습니다. 컨트랙트 계정 `eosio`가 인수 `{args...}`로 `eosio`에 정의된 액션 `setcode`를 실행함.

```
#         ${executor} <= ${contract}:${action} ${args...}
> console output from this execution, if any
```

아래에 보게 됩니다만, 액션은 다수의 컨트랙트로 실행될 수 있습니다.

커맨드 마지막 부분에 사용한 `-p eosio`는 `cleos`에게 `eosio` 계정의 권한을 사용하여 액션을 실행하라고 알려줍니다. 그러면 `cleos`는 앞서 지갑에 담았던 `eosio` 계정의 비밀키를 사용합니다.


## 계정 생성

이제 베이직 시스템 컨트랙트를 셋업했기 때문에 계정을 만들 수 있습니다. `user`, `tester`라는 계정 두 개를 만들 계획이고, 각 계정마다 비밀키/공개키가 필요합니다. 튜토리얼에서는 이 두 개의 계정에 동일한 비밀키/공개키를 사용하겠습니다.

먼저 두 계정에 사용할 키를 생성합니다.

```
$ cleos create key
Private key: 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
Public key: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
```

이어서 생성한 키를 지갑에 담습니다.
```
$ cleos wallet import 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
imported private key for: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
```

**주의** 위의 예시에 나온 키를 사용하지 말고, 실제로 `cleos`를 실행했을 때 나오는 키를 사용해야 합니다!

키는 자동으로 지갑에 추가되지 않습니다. 이 단계를 건너뛰면 계정에 대한 컨트롤을 잃게 됩니다.

## 유저 계정 2개 생성

이제 위에서 생성한 키를 이용하여 `user`와 `tester`, 이렇게 2개의 계정을 생성하겠습니다.

```
$ cleos create account eosio user  EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 8aedb926cc1ca31642ada8daf4350833c95cbe98b869230f44da76d70f6d6242  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...

$ cleos create account eosio tester EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083 366 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
```

**주의** `create account` 커맨드의 인수로 키 두 개가 필요합니다. 앞엣것은 (프러덕션 환경에서 매우 엄중한 보안 아래에 보관해야 할) OwnerKey이고, 뒤엣것은 ActiveKey입니다. 튜토리얼에서는 편의상 구분 없이 사용합니다.

우리가 `eosio::account_history_api_plugin`를 사용하고 있기 때문에 생성한 공개키로 제어할 수 있는 모든 계정을 확인할 수 있습니다.

```
$ cleos get accounts EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
{
  "account_names": [
    "tester",
    "user"
  ]
}
```


## 토큰 컨트랙트 생성

이 단계에서는 블록체인만으로는 할 수 있는 게 별로 없기 때문에, `eosio.token`이라는 컨트랙트를 배포합시다. 이 컨트랙트 하나만 배포해놓으면, 여러 사용자들이 이 컨트랙트 하나만 이용해서 서로 다른 토큰을 만들 수 있습니다.

토큰 컨트랙트를 배포하려면, 컨트랙트를 배포할 계정을 만들어야 합니다.

```
$ cleos create account eosio eosio.token  EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
...
```

이제 `${EOSIO_SOURCE}/build/contracts/eosio.token`에 있는 컨트랙트를 배포합니다.

```
$ cleos set contract eosio.token build/contracts/eosio.token -p eosio.token
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 528bdbce1181dc5fd72a24e4181e6587dace8ab43b2d7ac9b22b2017992a07ad  8708 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d0100000001ce011d60067f7e7f7f7f7f00...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```

### EOS 토큰 생성

`contracts/eosio.token/eosio.token.hpp`에 정의된 `eosio.token` 인터페이스를 살펴봅시다.
```
   void create( account_name issuer,
                asset        maximum_supply,
                uint8_t      can_freeze,
                uint8_t      can_recall,
                uint8_t      can_whitelist );


   void issue( account_name to, asset quantity, string memo );

   void transfer( account_name from,
                  account_name to,
                  asset        quantity,
                  string       memo );
```

새로운 토큰을 생성하려면 올바른 인자로 `create(...)` 액션을 호출해야 합니다. 이 커맨드는 다른 토큰과 구별하기 위해 토큰의 최대 공급량 수치에 토큰 심볼을 덧붙입니다. 발행자는 발행을 실행하고, 토큰 소유자를 동결, 회수, 허용하는 액션을 실행하는 권한을 가진 사용자가 됩니다.

위치 매개변수(positional argument)를 사용하여 간결하게 커맨드를 실행해봅니다.

```
$ cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOS", 0, 0, 0]' -p eosio.token
executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  260 bytes  1000 cycles
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whitelis...
```

또는, named argument로 자세하게 호출하는 방법도 있습니다.

```
$ cleos push action eosio.token create '{"issuer":"eosio", "maximum_supply":"1000000000.0000 EOS", "can_freeze":0, "can_recall":0, "can_whitelist":0}' -p eosio.token
executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  260 bytes  1000 cycles
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whitelis...
```


이 커맨드는 `EOS`라는 심볼을 갖는 새로운 토큰을 생성합니다. 이 토큰은 소수점 4자리까지 지원하고 최대 공급량은 1000000000.0000 EOS입니다.

이 토큰을 생성하기 위하여 `eosio.token` 컨트랙트의 권한이 필요합니다. `eosio.token` 컨트랙트가 `EOS`와 같은 토큰 심볼 네임스페이스를 "소유"하기 때문입니다. 여러 주체들이 자동으로 토큰 심볼 구매가 가능하도록, 앞으로 eosio.token 컨트랙트가 변경될 수 있습니다. 그래서 위의 액션을 실행하려면 커맨드에 `-p eosio.token`를 붙여야 합니다.

### `user` 계정에 토큰 발행

이제 토큰을 만들었으니, 발행자는 새로운 토큰들을 앞서 만든 `user` 계정으로 발행할 수 있습니다.

우리는 (named argument 방식 대신)  위치 매개변수 방식을 사용하겠습니다.

```
$ cleos push action eosio.token issue '[ "user", "100.0000 EOS", "memo" ]' -p eosio
executed transaction: 822a607a9196112831ecc2dc14ffb1722634f1749f3ac18b73ffacd41160b019  268 bytes  1000 cycles
#   eosio.token <= eosio.token::issue           {"to":"user","quantity":"100.0000 EOS","memo":"memo"}
>> issue
#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 EOS","memo":"memo"}
>> transfer
#         eosio <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 EOS","memo":"memo"}
#          user <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 EOS","memo":"memo"}
```

출력 내용을 살펴보면 다양한 액션이 눈에 띕니다. 발행(issue) 1건과 이체(transfer) 3건이 확인됩니다. 우리가 적당한 권한으로 실행한 유일한 액션은 발행(issue) 하나입니다만, `issue` 액션은 "inline transfer"를 수행했고, "inline transfer"는 송신자 계정과 수신자 계정에 각각 이체 정보를 전달했습니다. 출력 내용은, 호출된 액션 핸들러 전체 목록과 호출된 순서와 액션으로 생성된 출력이 있는지 없는지를 보여줍니다.

사실 `eosio.token` 컨트랙트는 `inline transfer`를 건너뛰고 밸런스를 직접적으로 조절할 수 있었지만, 이 경우에는 `eosio.token` 컨트랙트가 모든 계정 잔액이 이체 액션의 합으로 계산이 되어야 한다는 토큰 규칙을 따랐습니다. 또, 입금과 출금을 자동적으로 처리할 수 있도록 송신자와 수신자 모두에게 정보를 이체 정보를 전달해야 하는 토큰 규칙도 지켰습니다.

네트워크에 브로드캐스트된 실제 트랜잭션 내용을 보고 싶다면, `-d -j` 옵션을 사용하세요. 이 옵션을 사용하면 브로드캐스트 하지 않고 트랜잭션 내용을 JSON 포맷으로 출력합니다.

```
$ cleos push action eosio.token issue '["user", "100.0000 EOS", "memo"]' -p eosio -d -j
{
  "expiration": "2018-04-01T15:20:44",
  "region": 0,
  "ref_block_num": 42580,
  "ref_block_prefix": 3987474256,
  "net_usage_words": 21,
  "kcpu_usage": 1000,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio.token",
      "name": "issue",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "00000000007015d640420f000000000004454f5300000000046d656d6f"
    }
  ],
  "signatures": [
    "EOSJzPywCKsgBitRh9kxFNeMJc8BeD6QZLagtXzmdS2ib5gKTeELiVxXvcnrdRUiY3ExP9saVkdkzvUNyRZSXj2CLJnj7U42H"
  ],
  "context_free_data": []
}
```

### `tester`계정으로 토큰 이체

이제 계정 `user`의 잔고에 토큰이 들어가 있으니, `tester` 계정으로 이체하겠습니다. `user`가 이 액션을 허가한다 의미로 권한 인자 `-p user`를 사용합니다.

```
$ cleos push action eosio.token transfer '[ "user", "tester", "25.0000 EOS", "m" ]' -p user
executed transaction: 06d0a99652c11637230d08a207520bf38066b8817ef7cafaab2f0344aafd7018  268 bytes  1000 cycles
#   eosio.token <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}
>> transfer
#          user <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}
#        tester <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}
```


## Hello World 컨트랙트

다음 단계로 우리의 첫 "hello world" 컨트랙트를 만들어 봅시다. "hello"라는 디렉토리를 만들고 `cd`로 디렉토리로 들어간 후 "hello.cpp" 파일을 만들어 아래 내용을 추가합니다.

#### hello/hello.cpp
```
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>
using namespace eosio;

class hello : public eosio::contract {
  public:
      using contract::contract;

      /// @abi action 
      void hi( account_name user ) {
         print( "Hello, ", name{user} );
      }
};

EOSIO_ABI( hello, (hi) )
```

이제 컴파일해서 web assembly(.wast)를 만듭니다.
```
$ eosiocpp -o hello.wast hello.cpp
```

이어서 abi를 만듭니다.

```
$ eosiocpp -g hello.abi hello.cpp
Generated hello.abi
```

이제 계정을 만들고 컨트랙트를 업로드합니다.

```
$ cleos create account eosio hello.code EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
...
$ cleos set contract hello.code ../hello -p hello.code
...
```

이제 컨트랙트를 실행할 수 있습니다.

```
$ cleos push action hello.code hi '["user"]' -p user
executed transaction: 4c10c1426c16b1656e802f3302677594731b380b18a44851d38e8b5275072857  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"user"}
>> Hello, user
```

위의 `hello/hello.cpp`로 구현한 컨트랙트는 누구나 사용할 수 있습니다. 다른 권한을 사용해보겠습니다.

```
$ cleos push action hello.code hi '["user"]' -p tester
executed transaction: 28d92256c8ffd8b0255be324e4596b7c745f50f85722d0c4400471bc184b9a16  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"user"}
>> Hello, user
```

위 커맨드의 `-p tester`에 사용한 `tester`는 권한 계정이고, `user`는 단순히 인자 하나입니다. 컨트랙트 실행시 자신이 가진 권한 계정으로 인증하려면, 컨트랙트를 이렇게 바꿔야 합니다.

"hello.cpp" 파일의 hi() 함수를 이렇게 바꿉니다.
```
void hi( account_name user ) {
   require_auth( user );
   print( "Hello, ", name{user} );
}
```
wast 파일을 컴파일하고 ABI를 만드는 과정을 다시 진행하고, `set contract`를 다시 실행하면 변경된 내용이 배포됩니다.

일부러 현재의 권한과 사용하는 권한을 다르게 설정하여 컨트랙트 액션을 실행하면 에러가 발생합니다.
```
$ cleos push action hello.code hi '["tester"]' -p user
Error 3030001: missing required authority
Ensure that you have the related authority inside your transaction!;
If you are currently using 'cleos push action' command, try to add the relevant authority using -p option.
Error Details:
missing authority of tester
```

다음과 같이 `tester` 권한을 주면 정상적으로 동작합니다.

```
$ cleos push action hello.code hi '["tester"]' -p tester
executed transaction: 235bd766c2097f4a698cfb948eb2e709532df8d18458b92c9c6aae74ed8e4518  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"tester"}
>> Hello, tester
```


## 거래소 컨트랙트 배포하기
위의 예제와 비슷하게, 거래소 컨트랙트를 배포하겠습니다. EOSIO 소프트웨어 루트에서 실행하는 커맨드는 아래와 같습니다.

```
$ cleos create account eosio exchange  EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 4d38de16631a2dc698f1d433f7eb30982d855219e7c7314a888efbbba04e571c  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"exchange","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxe...

$ cleos set contract exchange build/contracts/exchange -p exchange
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 5a63b4de8a1da415590778f163c5ed26dc164c960185b20fd834c297cf7fa8f4  35172 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"exchange","vmtype":0,"vmversion":0,"code":"0061736d0100000001f0023460067f7e7f7f7f7f00600...
#         eosio <= eosio::setabi                {"account":"exchange","abi":{"types":[{"new_type_name":"account_name","type":"name"}],"structs":[{"n...
```


## 번역 정보

이 문서는 아래의 위키 영문 원본을 번역했습니다.
* [Getting Started With Contracts](https://github.com/eosio/eos/wiki/Tutorial-Getting-Started-With-Contracts)
* 번역한 위키 원본 리비전 : 669211e579aa1fef575a0cb1ea0e6d14ced3371f
* 튜토리얼 원본은 EOSIO `eos` 저장소에 [TUTORIAL.md](https://github.com/EOSIO/eos/blob/master/TUTORIAL.md) 파일로 있었다가 EOSIO Dawn 3.0 출시와 함께 내용의 변경 없이 한국 표준시로 2018년 4월 6일에 리비전 669211e579aa1fef575a0cb1ea0e6d14ced3371f로 위 주소의 위키로 옮겨졌습니다.
