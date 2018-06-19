## 컨트랙트 시작하기

이 튜토리얼은 로컬 블록체인을 구성하고 스마트 컨트랙트를 테스트하는 방법을
다룹니다. 이 튜토리얼의 첫 번째 파트는 다음과 같이 구성되어 있습니다.

- 프라이빗 블록체인 시작하기
- 지갑 만들기
- BIOS 컨트랙트 로딩하기
- 계정 만들기

두 번째 파트는 스마트 컨트랙트를 만들고 블록체인에 배포하는 방법을 설명합니다.

- `eosio.token` 컨트랙트
- 거래소 컨트랙트
- Hello World 컨트랙트

이 튜토리얼을 진행하기 위해서는 EOSIO 소프트웨어가 설치되어 있어야 하고
`nodeos`와 `cleos`가 PATH 환경변수에 등록된 디렉토리에 설치되어 있어야 합니다.

(역자 주: `eosio`를 빌드하면 build/programs 디렉토리 아래에
`nodeos`와 `cleos` 디렉토리 아래 실행 바이너리가 있습니다.
상대 경로를 사용해도 무방합니다.)


## 프라이빗 블록체인 시작하기

다음 한 줄의 명령어로 하나의 node로만 실행되는 프라이빗 블록체인을
만들 수 있습니다.  

```
$ nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs

```

명령어 가장 앞쪽의 `nodeos`는 node를 실행합니다. 이 명령어는 튜토리얼 진행에
필요한 두 가지 플래그를 사용하고, 세 가지 플러그인을 로딩합니다.
위 명령어가 문제없이 실행이 되었다면 0.5초마다 블록이 생성되는 로그 메시지를
확인할 수 있습니다.

```
...
3165501ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00000a4c898956e0... #2636 @ 2018-05-25T16:52:45.500 signed by eosio [trxs: 0, lib: 2635, confirmed: 0]
3166004ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00000a4d2d4a5893... #2637 @ 2018-05-25T16:52:46.000 signed by eosio [trxs: 0, lib: 2636, confirmed: 0]
```

이 로그는 다음 세 가지 정보를 알려줍니다. 1. 방금 만든 블록체인이 라이브
상태이다. 2. 블록이 잘 생성되고 있다. 3. 블록체인을 사용할 수 있는 상태이다.

`nodeos` 명령어의 더 많은 정보를 얻으려면 다음 명령어를 사용하세요.

```
nodeos --help
```

## 지갑 만들기

지갑은 블록체인에서 액션을 승인할 때 필요한 비밀키들의 저장소입니다.
이 키들은 사용자를 위해 생성된 비밀번호를 이용해 암호화된 상태로 디스크에
 저장됩니다. 이 비밀키는 반드시 안전한 패스워드 관리 도구에 저장되어 있어야
합니다.

```
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub"
```

간단한 개발 환경의 용도로, 여러분의 지갑은 `nodeos`를 시작할 때 활성화했던
`eosio::wallet_api_plugin`을 통해 로컬 노드에 의해 관리되고 있습니다.
`nodeos`을 실행할 때마다, 지갑 안의 키들을 사용하기 위해선 "잠금 해제"
과정을 거쳐야 합니다.

```
$ cleos wallet unlock --password PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub
Unlocked: default
```

이렇게 지갑을 잠금 해제했습니다. 다만 비밀번호가 화면에 노출되죠? 실제로
명령어에 바로 비밀번호를 사용하면 bash 쉘 로그에 남기 때문에 보안상 권장되는
방식은 아닙니다. 따라서 보통 다음과 같은 방식을 사용합니다.

```
$ cleos wallet unlock
password:
```

지갑을 사용하지 않을 때는 지갑을 다시 잠그는 것이 보안에 좋습니다. `nodeos`를 종료하지 않으면서 지갑을 잠그려면 다음 명령어를 실행하면 됩니다.

```
$ cleos wallet lock
Locked: default
```

남은 튜토리얼을 진행하려면 지갑을 잠금 해제한 상태로 두면 됩니다.

## BIOS 컨트랙트 불러오기

이제 지갑에 `eosio` 계정의 비밀키를 담았기 때문에 초기 시스템 컨트랙트를
설정할 수 있습니다. 개발 용도로 디폴트 `eosio.bios` 컨트랙트를 사용할 수
있습니다. 이 `eosio.bios` 컨트랙트를 이용하여 다른 계정의 자원 할당을 직접
제어하고 다른 권한 관련 api를 호출할 수 있습니다. 퍼블릭 블록체인에서는
`eosio.bios` 컨트랙트를 통해 일반 컨트랙트의 메모리, 네트워크 및 CPU 활동을
위한 대역폭을 예약할 수 있는 토큰의 staking과 unstaking을 관리할 수 있습니다.

`eosio.bios` 컨트랙트는 EOSIO 소스 코드의 `contracts/eosio.bios` 디렉토리에
있습니다. `eosio` 소스의 루트 경로에서 `eosio.bios`를 실행시킬 때 아래의
명령어를 사용할 수 있습니다. 만약 다른 위치에서 `eosio.bios`를 실행하고
싶다면 `eosio.bios`의 절대 경로를 입력하면 됩니다.

```
$ cleos set contract eosio build/contracts/eosio.bios -p eosio
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

`cleos`는 두 가지 액션(`eosio::setcode`, `eosio::setabi`)을 실행하는 트랜잭션
하나를 만듭니다.

코드는 컨트랙트 실행 로직을 정의하고, ABI는 인수를 JSON과 바이너리 사이의
변환 방식을 정의합니다. 기술적으로 보자면 ABI 정의는 꼭 해야 하는 작업은
아니지만, EOSIO 유틸리티는 사용 편의를 위해서 ABI 정의가 필요합니다.

트랜잭션을 실행할 때마다 다음과 같은 형태의 출력을 볼 수 있습니다.

```
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

풀이하면 이렇습니다. 컨트랙트 계정 `eosio`가 인수 `{args...}`로 `eosio`에
정의된 액션 `setcode`를 실행함.

```
#         ${executor} <= ${contract}:${action} ${args...}
> console output from this execution, if any
```

아래에 보게 됩니다만, 액션은 다수의 컨트랙트로 실행될 수 있습니다.

커맨드 마지막 부분에 사용한 `-p eosio`는 `cleos`에게 `eosio` 계정의 권한을
사용하여 액션을 실행하라고 알려줍니다. 그러면 `cleos`는 앞서 지갑에 담았던
`eosio` 계정의 비밀키를 사용합니다.

## 계정 생성

이제 베이직 시스템 컨트랙트를 셋업했기 때문에 계정을 만들 수 있습니다.
`user`, `tester`라는 계정 두 개를 만들 계획이고, 각 계정마다 비밀키/공개키가
필요합니다. 튜토리얼에서는 이 두 개의 계정에 동일한 비밀키/공개키를
사용하겠습니다.

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

**주의** 위의 예시에 나온 키를 사용하지 말고, 실제로 `cleos`를 실행했을 때
나오는 키를 사용해야 합니다!

키는 자동으로 지갑에 추가되지 않습니다. 이 단계를 건너뛰면 계정에 대한
컨트롤을 잃게 됩니다.

## 유저 계정 2개 생성

이제 위에서 생성한 키를 이용하여 `user`와 `tester`, 이렇게 2개의 계정을 생성하겠습니다.

```
$ cleos create account eosio user EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 8aedb926cc1ca31642ada8daf4350833c95cbe98b869230f44da76d70f6d6242  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...

$ cleos create account eosio tester EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083 366 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
```
**주의** `create account` 커맨드의 인수로 키 두 개가 필요합니다.
앞엣것은 (프러덕션 환경에서 매우 엄중한 보안 아래에 보관해야 할) OwnerKey이고,
뒤엣것은 ActiveKey입니다. 튜토리얼에서는 편의상 구분 없이 사용합니다.

우리가 `eosio::account_history_api_plugin`를 사용하고 있기 때문에 생성한
공개키로 제어할 수 있는 모든 계정을 확인할 수 있습니다.

```
$ cleos get accounts EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
{
  "account_names": [
    "tester",
    "user"
  ]
}
```

## 이어서 - `eosio.token`, 거래소, `eosio.msig` 컨트랙트

이어서 다음 튜토리얼을 진행할 준비가 되었습니다. [eosio.token, 거래소, eosio.msig 컨트랙트](https://github.com/eoseoul/docs/blob/master/ko/translations/Tutorial-eosio-token-Contract.md)

# 번역 정보

* 원문 : https://github.com/eosio/eos/wiki/Tutorial-Getting-Started-With-Contracts
* 번역 기준 리비전 : 0c18223ea118597c7b2476118091c4094be6af99
