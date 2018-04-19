# EOSIO 스마트 컨트랙트

- [EOSIO 스마트 컨트랙트 들어가기](#EOSIO-스마트-컨트랙트-들어가기)
  - [필요한 사전 지식](#필요한-사전-지식)
  - [EOSIO 스마트 컨트랙트 기본](#EOSIO-스마트-컨트랙트-기본)
- [스마트 컨트랙트 파일](#스마트-컨트랙트-파일)
  - [hpp](#hpp)
  - [cpp](#cpp)
  - [wast](#wast)
  - [abi](#abi)
- [스마트 컨트랙트 디버깅하기](#스마트-컨트랙트-디버깅하기)
  - [방법](#방법)
  - [Print](#Print)
  - [예제](#예제)
- [번역 정보](#번역-정보)

## EOSIO 스마트 컨트랙트 들어가기

### 필요한 사전 지식

**C / C++ 경험**

EOSIO 기반 블록체인은 [WebAssembly](http://webassembly.org/) (WASM)를 이용한
사용자 생성 응용 프로그램과 코드를 실행합니다. WASM은 Google, Microsoft, Apple
및 다른 기업들로부터 광범위한 지원을 받는 새로운 웹 표준입니다. 현재 WASM으로
컴파일되는 응용 프로그램을 빌드하는 데있어 가장 성숙한 툴체인은 
[clang/llvm](https://clang.llvm.org/)의 C/C++ 컴파일러입니다.

다른 개발 주체들이 만들고 있는 툴체인은 Rust, Python, Solidity가 있습니다.
이 언어들이 더 단순해 보일 수도 있지만, 각 언어마다의 성능은 작성하려는
응용 프로그램의 규모에 영향을 줄 수 있습니다. 고성능 보안 스마트 
컨트랙트를 작성하고 C++를 가까운 미래에 사용할 계획인 경우, C++가 제일 좋은
언어가 될 것으로 기대합니다.

**Linux / Mac OS 경험**

EOSIO는 다음 환경을 지원합니다:
- Amazon 2017.09 및 그 이상 버전
- Centos 7
- Fedora 25 및 그 이상 버전 (Fedora 27 추천)
- Mint 18
- Ubuntu 16.04 (Ubuntu 16.10 추천)
- MacOS Darwin 10.12 및 그 이상 버전 (MacOS 10.13.x 추천)

**커맨드 라인 지식**

EOSIO와 함께 다양한 툴이 제공됩니다. 이 툴을 사용하려면 커맨드 라인으로 
명령어를 이용할 수 있는 기본적인 지식이 필요합니다.

### EOSIO 스마트 컨트랙트 기본

**통신 모델**

EOSIO 스마트 컨트랙트는 액션과 공유 메모리 DB 접근을 통해 서로 통신합니다.
이를테면 하나의 컨트랙트는 해당 컨트랙트가 비동기적 방식 트랜잭션의 read
scope에 포함되어 있다면 다른 컨트랙트의 DB 상태를 읽을 수 있습니다.
비동기 통신은 자원 제한 알고리즘이 해결해야 하는 스팸을 초래할 수 있습니다.
컨트랙트 안에서 정의된 두 가지 통신 모드를 살펴보겠습니다.

- **Inline**. 인라인 액션은 현재 트랜잭션 내에서 실행되거나 unwind됨을 보장합니다. 성공과 실패에 무관하게 알림(notification)은 전달되지 않습니다. 인라인은 해당 인라인을 실행한 부모 트랜잭션이 가진 scope와 권한을 그대로 이어받아 동작합니다.

- **Deferred**. 지연 액션은 블록 프로듀서의 재량에 따라 실행 스케줄이 정해집니다. 통신 결과를 전달하거나 그냥 만료될 수도 있습니다. 지연 액션은 다른 scope까지 접근할 수 있고 지연 액션을 보낸 컨트랙트의 권한을 그대로 사용할 수 있습니다.

**액션 vs 트랜잭션**

액션은 단일 실행 단위입니다. 이와 비교하면, 트랜잭션은 하나 이상 액션의
모음입니다. 컨트랙트와 계정은 액션의 형태로 통신합니다. 액션은 하나씩 보낼 수
있고, 한꺼번에 실행해야 할 경우 합쳐서 보낼 수도 있습니다.

*액션 하나로 구성된 트랜잭션*.

```base
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
    ""
  ],
  "context_free_data": []
}
```

*여러 개의 액션으로 구성된 트랜잭션*, 이 액션들은 모두 성공하거나 모두 실패해야 합니다.
```base
{
  "expiration": "...",
  "region": 0,
  "ref_block_num": ...,
  "ref_block_prefix": ...,
  "net_usage_words": ..,
  "kcpu_usage": ..,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }, {
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }
  ],
  "signatures": [
    ""
  ],
  "context_free_data": []
}
```

**액션 이름 제한**

액션 타입은 실제로 **base32로 인코딩한 64-bit 정수형**입니다. 즉, 
앞 12개의 문자는 a-z, 1-5, '.'로 제한됩니다. 만약 13번째 문자가 있다면
앞 16개의 문자는 '.'과 a-p로 제한됩니다.

**트랜잭션 승인**

트랜잭션 해시를 받았다고 하더라도 트랜잭션이 승인되었다는 뜻은 아닙니다.
단지 노드가 트랜잭션을 에러 없이 수용했다는 뜻이고, 다른 프로듀서가 해당
트랜잭션을 수용할 확률 역시 높다는 뜻입니다.

승인이 되어야 트랜잭션이 포함된 블록 넘버를 통해 트랜잭션 이력 속에서 
트랜잭션을 확인할 수 있습니다.

## 스마트 컨트랙트 파일

단순한 개발을 위해서 **[eosiocpp](https://github.com/EOSIO/eos/wiki/Programs-&-Tools#eosiocpp)** 라는 툴을 만들었습니다. 이 툴을 이용해서 새로운 스마트
컨트랙트를 개발하기 위한 기본 틀을 만들 수 있습니다. `eosiocpp`는 
개발 시작을 위한 기본 틀로 3개의 스마트 컨트랙트 파일을 만들어줍니다.

```base
$ eosiocpp -n ${contract}
```

위와 같이 실행하면 `./${project}` 폴더에 기본 틀이 되는 아래 3개의 파일을 만듭니다.
```base
${contract}.abi ${contract}.hpp ${contract}.cpp
```

### hpp

`${contract}.hpp`는 `.cpp` 파일에서 참조하는 변수, 상수, 함수를 담는 헤더 파일입니다.

### cpp

`${contract}.cpp` 파일은 컨트랙트의 함수를 담는 소스 코드입니다.

`eosiocpp`로 `.cpp` 파일을 생성하면, 생성된 `.cpp` 파일은 아래와 같은 내용일 것입니다.

```base
#include <${contract}.hpp>

/**
 *  The init() and apply() methods must have C calling convention so that the blockchain can lookup and
 *  call these methods.
 */
extern "C" {

    /**
     *  This method is called once when the contract is published or updated.
     */
    void init()  {
       eosio::print( "Init World!\n" ); // Replace with actual code
    }

    /// The apply method implements the dispatch of actions to this contract
    void apply( uint64_t code, uint64_t action ) {
       eosio::print( "Hello World: ", eosio::name(code), "->", eosio::name(action), "\n" ); 
    }

} // extern "C"
```

이 예제에서는 `init`과 `apply` 함수만 보일 것입니다. 이 두 함수는 전달된 액션에 대한 로그를 남기고 다른 체크는 하지 않습니다. 블록 프로듀서가 허용하는 한 누구나 어떤 액션도 보낼 수 있습니다. 요구되는 서명도 없어서 컨트랙트는 사용한 대역폭만큼 과금됩니다.

**init**

`init` 함수는 배포 처음 한 번만 실행될 것입니다. 예를 들어 환전 컨트랙트를 위한 토큰 공급량과 같은 컨트랙트 변수를 초기화하는 데 사용합니다.

**apply**

`apply`는 액션 핸들러입니다. 모든 액션을 지켜보고 있다가 함수 내에 정의된 로직에 따라서 반응합니다. `apply` 함수는 `code`와 `action`이라는 두 개의 파라미터 입력이 필요합니다.

**code filter**

특정 컨트랙트에 반응하기 위한 `apply` 함수의 구조는 아래와 같습니다. code filter를 생략하면 일반적인 컨트랙트에 반응하도록 구성할 수도 있습니다.

```base
if (code == N(${contract_name}) {
    // your handler to respond to particular action
}
```

코드 블럭 안에서 각각의 액션에 대한 응답을 정의할 수도 있습니다.

**action filter**

특정 액션에 반응하기 위한 `apply` 함수의 구조는 아래와 같습니다. 액션 필터는 보통 코드 필터와 결합하여 사용합니다.

```base
if (action == N(${action_name}) {
    //your handler to respond to a particular action
}
```

### wast

EOSIO 블록체인에 배포되는 모든 프로그램은 WASM 포맷으로 컴파일해야 합니다.
블록체인이 받아들이는 유일한 포맷입니다.

CPP 파일이 준비된 상태가 되면, `eosiocpp` 툴로 WASM의 텍스트 버전(.wast)으로
컴파일할 수 있습니다.

```base
$ eosiocpp -o ${contract}.wast ${contract}.cpp
```
### abi

Application Binary Interface(ABI)는 사용자 액션을 JSON과 바이너리 형태 사이에서변환하는 정의입니다. ABI는 DB 상태를 JSON으로 변환하는 방식을 정의합니다.
ABI로 컨트랙트를 정의해놓으면 개발자와 사용자는 JSON으로 컨트랙트와 부드럽게 통신할 수 있을 것입니다.

ABI 파일은 `eosiocpp` 툴로 `.hpp` 파일을 이용해서 생성합니다.

```base
$ eosiocpp -g ${contract}.abi ${contract}.hpp
```

아래는 껍데기 컨트랙트 ABI의 예제입니다.
```base
{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
      "name": "account",
      "base": "",
      "fields": {
        "account": "name",
        "balance": "uint64"
      }
    }
  ],
  "actions": [{
      "action": "transfer",
      "type": "transfer"
    }
  ],
  "tables": [{
      "table": "account",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["account"],
      "key_types" : ["name"]
    }
  ]
}
```

이 ABI를 보면 `transfer` 타입의 `transfer` 액션의 정의를 볼 수 있습니다.
이 명세를 통해서 EOSIO는 `${account}->transfer` 액션이 보이면 전달된 데이터는
`transfer` 타입이라는 사실을 알 수 있습니다. `transfer` 타입은 `structs`
배열 속 `name`이 `transfer`로 설정된 객체로 정의되어 있습니다.

```base
...
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
...
```

ABI에는 `from`, `to`, `quantity`라는 필드가 있습니다. 이 필드는 각각 `account_name`, `uint64` 타입으로 정의되어 있습니다. `account_name`는 `uint64`로 base32 문자열을 표현하는 built-in 타입입니다. 사용할 수 있는 built-in 타입 정보는 [다음 코드](https://github.com/EOSIO/eos/blob/master/libraries/chain/contracts/abi_serializer.cpp)를 참고하세요.

```base
{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
...
```

위 `types` 배열에서는 앞서 정의한 타입의 alias 목록을 정의합니다. 즉, `account_name`의 alias로 `name`을 정의했습니다.

## 스마트 컨트랙트 디버깅하기

스마트 컨트랙트를 디버그하기 위해서 로컬 `nodeos` 노드를 셋업해야 합니다. 개별적으로 동작하는 프라이빗 테스트넷로 로컬 `nodeos` 노드를 셋업하거나 퍼블릭 테스트넷(혹은 공식 테스트넷)을 연결한 노드를 셋업해서 사용할 수도 있습니다.

처음 스마트 컨트랙트를 작성할 때 프라이빗 테스트에서 스마트 컨트랙트를 테스트하고 디버그하기를 권합니다. 왜나하면 블록체인 전체를 완전히 제어할 수 있기 때문입니다. 그렇게 하면 필요한 EOS를 무제한으로 확보할 수 있고 필요할 때마다 블록체인 상태를 리셋할 수 있습니다. 프러덕션에 올릴 준비가 되면 로컬 `nodeos`를 퍼블릭 테스트넷(또는 공식 테스트넷)에 연결해서 테스트를 진행하면 로컬 `nodeos`에서 테스트넷의 로그를 확인할 수 있습니다.

컨셉은 같습니다. 아래 가이드에서는 프라이빗 테스트넷에서 디버깅하는 방법을 다루겠습니다.

로컬 `nodeos`를 아직 셋업하지 않았다면 [로컬 셋업 가이드](https://github.com/eoseoul/docs/blob/master/ko/translations/Local-Environment.md)를 따라하세요. 
[퍼블릭 테스트넷(또는 공식 테스트넷) 연결 가이드](https://github.com/eosio/eos/wiki/Testnet%3A%20Public)를 따라서 `config.ini` 파일을 수정하지 않았다면, 기본 설정된 로컬 `nodeos`는 프라이빗 테스트넷 모드로 동작할 것입니다.

### 방법
스마트 컨트랙트를 디버그하는 가장 주된 방법은 **동굴 디버깅(caveman debugging)**인데, 변수 값을 확인하고 컨트랙트의 흐름을 체크하기 위해 print 기능을 이용하는 방법입니다. 스마트 컨트랙트에서 print는 Print API ([C](https://github.com/EOSIO/eos/blob/master/contracts/eosiolib/print.h)와 [C++](https://github.com/EOSIO/eos/blob/master/contracts/eosiolib/print.hpp))를 사용하면 됩니다. C++ API는 C API wrapper입니다. 그래서 대부분 C++ API만 사용하겠습니다.

### Print
Print C API는 다음 데이터 타입을 지원합니다.
- prints - null 문자로 끝나는 char array (string)
- prints_l - 주어진 크기의 char array (string)
- printi - 64비트 unsigned integer
- printi128 - 128비트 unsigned integer
- printd - 64비트 unsigned integer로 인코딩된 double
- printn - 64비트 unsigned integer로 인코딩된 base32 문자열
- printhex - 바이너리 데이터로 주어진 HEX

Print C++ API는 위의 C API 일부를 print() 함수로 오버라이딩했고, 그래서 사용자는 사용하려는 필요에 맞는 print 함수를 결정할 필요가 없습니다. Print C++ API는 다음 데이터 타입을 지원합니다.
- null 문자로 끝나는 char array (string)
- integer (128-bit unsigned, 64-bit unsigned, 32-bit unsigned, signed, unsigned)
- 64비트 unsigned integer로 인코딩된 base32 문자열
- print() 메소드가 있는 struct

### 예제
이제 디버깅용 예제로 사용할 새로운 컨트랙트를 작성해봅시다.

- debug.hpp
```cpp
#include <eoslib/eos.hpp>
#include <eoslib/db.hpp>

namespace debug {
    struct foo {
        account_name from;
        account_name to;
        uint64_t amount;
        void print() const {
            eosio::print("Foo from ", eosio::name(from), " to ",eosio::name(to), " with amount ", amount, "\n");
        }
    };
}
```
- debug.cpp
```cpp
#include <debug.hpp>

extern "C" {

    void init()  {
    }

    void apply( uint64_t code, uint64_t action ) {
        if (code == N(debug)) {
            eosio::print("Code is debug\n");
            if (action == N(foo)) {
                 eosio::print("Action is foo\n");
                debug::foo f = eosio::current_message<debug::foo>();
                if (f.amount >= 100) {
                    eosio::print("Amount is larger or equal than 100\n");
                } else {
                    eosio::print("Amount is smaller than 100\n");
                    eosio::print("Increase amount by 10\n");
                    f.amount += 10;
                    eosio::print(f);
                }
            }
        }
    }
} // extern "C"
```
- debug.hpp
```cpp
{
  "structs": [{
      "name": "foo",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "amount": "uint64"
      }
    }
  ],
  "actions": [{
      "action_name": "foo",
      "type": "foo"
    }
  ]
}

```

이제 배포하고 메시지를 보내봅시다. `debug` 계정을 만들었고 해당 계정의 키를 지갑에 import했다고 가정하겠습니다.
```bash
$ eosiocpp -o debug.wast debug.cpp
$ cleos set contract debug debug.wast debug.abi
$ cleos push message debug foo '{"from":"inita", "to":"initb", "amount":10}' --scope debug
```

로컬 `nodeos` 로그를 확인하면, 위 명령어를 실행한 후 아래와 같은 로그를 확인할 수 있습니다.

```
Code is debug
Action is foo
Amount is smaller than 100
Increase amount by 10
Foo from inita to initb with amount 20
```
이렇게 해서 메시지가 원하는 흐름으로 제어되었고 금액도 제대로 업데이트되었다는 점을 확인할 수 있습니다. 위 메시지는 적어도 2번 이상 보게 되고, 이는 각 트랜잭션이 검증(verification), 블록 생성, 블록 적용에 이르는 단계마다 트랜잭션이 실행되기 때문이므로 정상입니다.

## 번역 정보

* 원문 : https://github.com/EOSIO/eos/wiki/Smart%20Contract
* 번역 기준 리비전 : 669211e579aa1fef575a0cb1ea0e6d14ced3371f
