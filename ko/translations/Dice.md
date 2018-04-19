주사위
-----------------

이 컨트랙트는 두 명의 플레이어가 50 대 50의 승률을 가진 간단한 주사위 게임이다.

플레이하기 전 모든 플레이어는 `@exchange` 컨트랙트와 마찬가지로 `@dice` 컨트랙트 계정에 자금을 입금한다.

1. 플레이어 1은 1 EOS를 걸고 SHA256(secret1) 으로 나온 해시값을 제출한다.
2. 플레이어 2는 1 EOS를 걸고 SHA256(secret2) 으로 나온 해시값을 제출한다.

플레이어 1과 2는 같은 금액을 걸었기 때문에, 주문 내역이 일치하고 이제 게임이 시작된다.

3. 플레이어 둘 중 어느 하나가 패를 오픈한다.
4. 5분 마감 시간이 시작되고, 다른 플레이어가 자신의 패를 오픈하지 않으면 먼저 오픈한 플레이어가 승리한다.
5. 다른 플레이어가 패를 오픈하면 SHA256( cat(secret1,secret2) ) 의 결과에 따라 승자가 결정되고 승리보수가 지급된다.
6. 마감시간이 지나면 둘 다 청구하여 보상을 수령할 수 있다.


인터페이스 제작자를 위한 인센티브
-----------------

이 게임을 변형해서 플레이어가 승리했을 때 제작자가 커미션을 받게 하도록 정보를 추가할 수도 있다. 이렇게 커미션이 있다면 서비스 제공 업체가 오래도록  적시에
게임을 계속 서비스할 재정적 인센티브가 주어질 뿐더러, 게임 퀄리티를 향상시키고
재미있는 인터페이스를 붙이게도 할 것이다.


다른 게임
-----------
이와 동일한 기초 모델로 더 탄탄한 게임을 만들 수 있다.


잠재적 취약성
-------
1. 블록 프로듀서가 패-오픈 트랜잭션을 거부할 수 있다.
2. 패자가 승자를 5분 동안 기다리게 만들 수 있다.
3. 서비스 제공자가 구현한 자동 패-오픈 시스템이 실행되지 않을 가능성이 있다.
4. 게임 도중에 인터넷이 끊길 수 있다.
5. 블록체인 재구성이 일어날 때, 패가 너무 빨리 오픈되는 등의 혼란이 발생할 수 있다.
    - `@dice`는 게임 생성이 비가역확정 상태에 도달하기까지(최대 45초) 패-오픈을 거절하여 사용자를 보호할 수 있다.
    - 사용자는 필요한 승인 횟수를 결정하여 리스크를 감수할 수 있다.
    - 작은 금액의 경우 아마 문제되지 않을 것이다.
    - DPOS 체인이 정상적으로 작동하는 경우 체인 재구성이 거의 없을 것이다.


`cleos`를 이용한 게임 세션 예제
-------
#### 사전 준비
* 지갑은 unlock 상태로 만들고 아래 비밀키를 사용한다.

	**5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3**
	**5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR**

##### BIOS 컨트랙트 로딩
````bash
cleos set contract eosio build/contracts/eosio.bios -p eosio
````

##### `eosio.token` 계정 생성
````bash
cleos create account eosio eosio.token EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
````

##### `eosio.token` 계정에 `eosio.token` 컨트랙트 코드를 배포
````bash
cleos set contract eosio.token build/contracts/eosio.token -p eosio.token
````

##### 주사위 컨트랙트 배포용 계정 생성
````bash
cleos create account eosio dice EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
````

##### `dice` 계정에 `dice` 컨트랙트 코드를 배포
````bash
cleos set contract dice build/contracts/dice -p dice
````

##### 기본 `EOS` 토큰 생성
````bash
cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOS", 0, 0, 0]' -p eosio.token
````

##### `alice` 계정 생성
````bash
cleos create account eosio alice EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
````

##### `bob` 계정 생성
````bash
cleos create account eosio bob EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
````

##### 1000 EOS 를 `alice` 계정에 발행
````bash
cleos push action eosio.token issue '[ "alice", "1000.0000 EOS", "" ]' -p eosio
````

##### 1000 EOS 를 `bob` 계정에 발행
````bash
cleos push action eosio.token issue '[ "bob", "1000.0000 EOS", "" ]' -p eosio
````

##### `dice` 컨트랙트 실행시 `alice` 계정 대신 입금과 이체를 할 수 있도록 권한 허가
````bash
cleos set account permission alice active '{"threshold": 1,"keys": [{"key": "EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4","weight": 1}],"accounts": [{"permission":{"actor":"dice","permission":"active"},"weight":1}]}' owner -p alice
````

##### `dice` 컨트랙트 실행시 `bob` 계정 대신 입금과 이체를 할 수 있도록 권한 허가
````bash
cleos set account permission bob active '{"threshold": 1,"keys": [{"key": "EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4","weight": 1}],"accounts": [{"permission":{"actor":"dice","permission":"active"},"weight":1}]}' owner -p bob
````

##### `alice` 계정에서 100 EOS 만큼 `dice` 컨트랙트로 이체 - 베팅용 자금
````bash
cleos push action dice deposit '[ "alice", "100.0000 EOS" ]' -p alice
````

##### `bob` 계정에서 100 EOS 만큼 `dice` 컨트랙트로 이체 - 베팅용 자금
````bash
cleos push action dice deposit '[ "bob", "100.0000 EOS" ]' -p bob
````

##### `alice` 계정이 사용할 랜덤 문자열 생성
````bash
openssl rand 32 -hex
28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905
````

##### `alice` 계정이 사용할 랜덤 문자열의 해시 생성 - sha256(secret)
````bash
echo -n '28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905' | xxd -r -p | sha256sum -b | awk '{print $1}'
d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883
````

##### `alice` 계정에서 3 EOS 베팅
````bash
cleos push action dice offerbet '[ "3.0000 EOS", "alice", "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883" ]' -p alice
````

##### `bob` 계정이 사용할 랜덤 문자열 생성
````bash
openssl rand 32 -hex
15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12
````

##### `bob` 계정이 사용할 랜덤 문자열의 해시 생성 - sha256(secret)
````bash
echo -n '15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12' | xxd -r -p | sha256sum -b | awk '{print $1}'
50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129
````

##### `bob` 계정에서도 3 EOS 베팅 (이제 게임 시작)
````bash
cleos push action dice offerbet '[ "3.0000 EOS", "bob", "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129" ]' -p bob
````

##### 게임이 시작된 직후의 `dice` 컨트랙트 테이블 상태 확인
````bash
cleos get table dice dice account
````
````json
{
  "rows": [{
      "owner": "alice",
      "eos_balance": "97.0000 EOS",
      "open_offers": 0,
      "open_games": 1
    },{
      "owner": "bob",
      "eos_balance": "97.0000 EOS",
      "open_offers": 0,
      "open_games": 1
    }
  ],
  "more": false
}
````

````bash
cleos get table dice dice game
````
````json
{
  "rows": [{
      "id": 1,
      "bet": "3.0000 EOS",
      "deadline": "1970-01-01T00:00:00",
      "player1": {
        "commitment": "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883",
        "reveal": "0000000000000000000000000000000000000000000000000000000000000000"
      },
      "player2": {
        "commitment": "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129",
        "reveal": "0000000000000000000000000000000000000000000000000000000000000000"
      }
    }
  ],
  "more": false
}
````

##### `bob` 계정이 자신의 패를 오픈(reveal)
````bash
cleos push action dice reveal '[ "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129", "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12" ]' -p bob
````

##### `bob` 계정이 패를 오픈한 직후의 게임 테이블(이제 `alice` 계정이 패를 오픈할 수 있는 deadline이 생김)
````bash
cleos get table dice dice game
````
````json
{
  "rows": [{
      "id": 1,
      "bet": "3.0000 EOS",
      "deadline": "2018-04-17T07:45:49",
      "player1": {
        "commitment": "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883",
        "reveal": "0000000000000000000000000000000000000000000000000000000000000000"
      },
      "player2": {
        "commitment": "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129",
        "reveal": "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12"
      }
    }
  ],
  "more": false
}
````

##### `alice` 계정이 패를 오픈 (승자가 결정되고, 게임이 종료됨)
````bash
cleos push action dice reveal '[ "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883", "28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905" ]' -p alice
````

##### 게임이 종료된 후 각 계정의 잔고
````bash
cleos get table dice dice account
````
````json
{
  "rows": [{
      "owner": "alice",
      "eos_balance": "103.0000 EOS",
      "open_offers": 0,
      "open_games": 0
    },{
      "owner": "bob",
      "eos_balance": "97.0000 EOS",
      "open_offers": 0,
      "open_games": 0
    }
  ],
  "more": false
}
````

##### `alice` 계정이 `dice` 컨트랙트에 묶인 103 EOS를 인출
````bash
cleos push action dice withdraw '[ "alice", "103.0000 EOS" ]' -p alice
````

##### 인출 이후 `alice`의 잔고
````bash
cleos get currency balance eosio.token alice eos
1003.0000 EOS
````

## 번역 정보

* 원문 : https://github.com/EOSIO/eos/blob/master/contracts/dice/README.md
* 번역 기준 리비전 : 93d5f9718ed68fe7fc139529459aa307a9d8c406
