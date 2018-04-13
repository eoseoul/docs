## `eosio.token`, 거래소, `eosio.msig` 컨트랙트

**이 튜토리얼은 튜토리얼 [컨트랙트 처음 시작하기](https://github.com/eoseoul/docs/blob/master/ko/translations/TUTORIAL.md)를 완료했다고 가정합니다.**

이 단계에서는 블록체인만으로는 할 수 있는 게 별로 없기 때문에,
`eosio.token`이라는 컨트랙트를 배포합시다. 이 컨트랙트 하나만 배포해놓으면,
여러 사용자들이 이 컨트랙트 하나만 이용해서 서로 다른 토큰을 만들 수 있습니다.

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

`contracts/eosio.token/eosio.token.hpp`에 정의된 `eosio.token` 인터페이스를
살펴봅시다.
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

새로운 토큰을 생성하려면 올바른 인자로 `create(...)` 액션을 호출해야 합니다.
이 커맨드는 다른 토큰과 구별하기 위해 토큰의 최대 공급량 수치에 토큰 심볼을
덧붙입니다. 발행자는 발행을 실행하고, 토큰 소유자를 동결, 회수, 허용하는
액션을 실행하는 권한을 가진 사용자가 됩니다.

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

이 커맨드는 `EOS`라는 심볼을 갖는 새로운 토큰을 생성합니다. 이 토큰은 소수점
4자리까지 지원하고 최대 공급량은 1000000000.0000 EOS입니다.

이 토큰을 생성하기 위하여 `eosio.token` 컨트랙트의 권한이 필요합니다.
`eosio.token` 컨트랙트가 `EOS`와 같은 토큰 심볼 네임스페이스를 "소유"하기
때문입니다. 여러 주체들이 자동으로 토큰 심볼 구매가 가능하도록, 앞으로
eosio.token 컨트랙트가 변경될 수 있습니다. 그래서 위의 액션을 실행하려면
커맨드에 `-p eosio.token`를 붙여야 합니다.

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

출력 내용을 살펴보면 다양한 액션이 눈에 띕니다. 발행(issue) 1건과
이체(transfer) 3건이 확인됩니다. 우리가 적당한 권한으로 실행한 유일한 액션은
발행(issue) 하나입니다만, `issue` 액션은 "inline transfer"를 수행했고,
"inline transfer"는 송신자 계정과 수신자 계정에 각각 이체 정보를 전달했습니다.
출력 내용은, 호출된 액션 핸들러 전체 목록과 호출된 순서와 액션으로 생성된
출력이 있는지 없는지를 보여줍니다.

사실 `eosio.token` 컨트랙트는 `inline transfer`를 건너뛰고 밸런스를 직접적으로
조절할 수 있었지만, 이 경우에는 `eosio.token` 컨트랙트가 모든 계정 잔액이
이체 액션의 합으로 계산이 되어야 한다는 토큰 규칙을 따랐습니다. 또, 입금과
출금을 자동적으로 처리할 수 있도록 송신자와 수신자 모두에게 정보를 이체 정보를
전달해야 하는 토큰 규칙도 지켰습니다.

네트워크에 브로드캐스트된 실제 트랜잭션 내용을 보고 싶다면, `-d -j` 옵션을
사용하세요. 이 옵션을 사용하면 브로드캐스트 하지 않고 트랜잭션 내용을
JSON 포맷으로 출력합니다.

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

이제 계정 `user`의 잔고에 토큰이 들어가 있으니, `tester` 계정으로
이체하겠습니다. `user`가 이 액션을 허가한다 의미로 권한 인자 `-p user`를
사용합니다.

```
$ cleos push action eosio.token transfer '[ "user", "tester", "25.0000 EOS", "m" ]' -p user
executed transaction: 06d0a99652c11637230d08a207520bf38066b8817ef7cafaab2f0344aafd7018  268 bytes  1000 cycles
#   eosio.token <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}
>> transfer
#          user <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}
#        tester <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}
```

## 거래소 컨트랙트 배포하기

위의 예제와 비슷하게 `exchange` 컨트랙트를 배포하겠습니다. `exchange` 컨트랙트는
통화를 만들고 거래할 수 있는 기능을 제공합니다. 아래 명령어는 EOSIO 소스 코드의
루트에서 실행할 수 있습니다.

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

## `eosio.msig` 컨트랙트 배포하기
`eosio.msig` 컨트랙트는 다수의 관계자가 단일 트랜잭션을 비동기적으로 서명할 수
있는 기능을 제공합니다. EOSIO는 기본적인 수준에서 다중 서명(멀티시그) 기능을
제공하지만, 데이터가 오고가고 서명되는 동기적 사이드 채널이 필요합니다.
`eosio.msig`는 비동기적인 제안과 승인, 다수 관계자의 동의를 얻은 트랜잭션을
네트워크에 퍼뜨리는 보다 사용자 친화적인 방법을 제공합니다.

아래와 같은 명령어로 `eosio.msig` 컨트랙트를 배포합니다.
```
$ cleos create account eosio eosio.msig  EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.msig","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJ...
  
$ cleos set contract eosio.msig build/contracts/eosio.msig -p eosio.msig
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: a113a7db8c878dfd894671792770b59a04efb3aa8295f5b3d585daf89c314ec9  8964 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d0100000001bd011b60047f7e7e7f0060047...
#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":{"types":[{"new_type_name":"account_name","type":"name"},{"new_type_na...
```

## 이어서 -  Hello World 튜토리얼

이제 다음 튜토리얼 [Hello World 튜토리얼](https://github.com/eoseoul/docs/blob/master/ko/translations/Tutorial-Hello-World-Contract.md) 로 넘어갑시다.

# 번역 정보

* 원문 : https://github.com/eosio/eos/wiki/Tutorial-eosio-token-Contract
* 번역 기준 리비전 : 0c18223ea118597c7b2476118091c4094be6af99
