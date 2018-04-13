## Hello World 컨트랙트
**이 튜토리얼은 튜토리얼 [컨트랙트 처음 시작하기](https://github.com/eoseoul/docs/blob/master/ko/translations/TUTORIAL.md)와 [eosio.token, 거래소, eosio.msig 컨트랙트](https://github.com/eoseoul/docs/blob/master/ko/translations/Tutorial-eosio-token-Contract.md)를 완료했다고 가정합니다.**

다음 단계로 우리의 첫 "hello world" 컨트랙트를 만들어 봅시다. "hello"라는
디렉토리를 만들고 `cd`로 디렉토리로 들어간 후 "hello.cpp" 파일을 만들어
아래 내용을 추가합니다.

### hello/hello.cpp
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
**주의** 컴파일러에서 에러 메시지가 나올 것입니다. 무시해도 됩니다.

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

위의 `hello/hello.cpp`로 구현한 컨트랙트는 누구나 사용할 수 있습니다.
다른 권한을 사용해보겠습니다.

```
$ cleos push action hello.code hi '["user"]' -p tester
executed transaction: 28d92256c8ffd8b0255be324e4596b7c745f50f85722d0c4400471bc184b9a16  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"user"}
>> Hello, user
```

위 커맨드의 `-p tester`에 사용한 `tester`는 권한 계정이고, `user`는 단순히
인자 하나입니다. 컨트랙트 실행시 자신이 가진 권한 계정으로 인증하려면,
컨트랙트를 이렇게 바꿔야 합니다.

"hello.cpp" 파일의 hi() 함수를 이렇게 바꿉니다.

```
void hi( account_name user ) {
   require_auth( user );
   print( "Hello, ", name{user} );
}
```
wast 파일을 컴파일하고 ABI를 만드는 과정을 다시 진행하고, `set contract`를
다시 실행하면 변경된 내용이 배포됩니다.

일부러 현재의 권한과 사용하는 권한을 다르게 설정하여 컨트랙트 액션을
실행하면 에러가 발생합니다.
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

## Hello World 리카도 컨트랙트

모든 스마트 컨트랙트는 스마트 컨트랙트가 준수하는 리카도 컨트랙트와 연결되어야
합니다. 리카도 컨트랙트는 스마트 컨트랙트의 각 액션과 관련된 법적 구속력있는
행동을 명시합니다. Hello World 컨트랙트에 대한 리카도 컨트랙트의 예시는
아래와 같습니다.

```markdown
## CONTRACT FOR HELLO WORLD

### Parameters
Input paramters: NONE

Implied parameters: 

* _**account_name**_ (name of the party invoking and signing the contract)

### Intent
INTENT. The intention of the author and the invoker of this contract is to print output. It shall have no other effect.

### Term
TERM. This Contract expires at the conclusion of code execution.

### Warranty
WARRANTY. {{ account_name }} shall uphold its Obligations under this Contract in a timely and workmanlike manner, using knowledge and recommendations for performing the services which meet generally acceptable standards set forth by EOS.IO Blockchain Block Producers.
  
### Default
DEFAULT. The occurrence of any of the following shall constitute a material default under this Contract: 

### Remedies
REMEDIES. In addition to any and all other rights a party may have available according to law, if a party defaults by failing to substantially perform any provision, term or condition of this Contract, the other party may terminate the Contract by providing written notice to the defaulting party. This notice shall describe with sufficient detail the nature of the default. The party receiving such notice shall promptly be removed from being a Block Producer and this Contract shall be automatically terminated. 
  
### Force Majeure
FORCE MAJEURE. If performance of this Contract or any obligation under this Contract is prevented, restricted, or interfered with by causes beyond either party's reasonable control ("Force Majeure"), and if the party unable to carry out its obligations gives the other party prompt written notice of such event, then the obligations of the party invoking this provision shall be suspended to the extent necessary by such event. The term Force Majeure shall include, without limitation, acts of God, fire, explosion, vandalism, storm or other similar occurrence, orders or acts of military or civil authority, or by national emergencies, insurrections, riots, or wars, or strikes, lock-outs, work stoppages, or supplier failures. The excused party shall use reasonable efforts under the circumstances to avoid or remove such causes of non-performance and shall proceed to perform with reasonable dispatch whenever such causes are removed or ceased. An act or omission shall be deemed within the reasonable control of a party if committed, omitted, or caused by such party, or its employees, officers, agents, or affiliates. 
  
### Dispute Resolution
DISPUTE RESOLUTION. Any controversies or disputes arising out of or relating to this Contract will be resolved by binding arbitration under the default rules set forth by the EOS.IO Blockchain. The arbitrator's award will be final, and judgment may be entered upon it by any court having proper jurisdiction. 
  
### Entire Agreement
ENTIRE AGREEMENT. This Contract contains the entire agreement of the parties, and there are no other promises or conditions in any other agreement whether oral or written concerning the subject matter of this Contract. This Contract supersedes any prior written or oral agreements between the parties. 

### Severability
SEVERABILITY. If any provision of this Contract will be held to be invalid or unenforceable for any reason, the remaining provisions will continue to be valid and enforceable. If a court finds that any provision of this Contract is invalid or unenforceable, but that by limiting such provision it would become valid and enforceable, then such provision will be deemed to be written, construed, and enforced as so limited. 

### Amendment
AMENDMENT. This Contract may be modified or amended in writing by mutual agreement between the parties, if the writing is signed by the party obligated under the amendment. 

### Governing Law
GOVERNING LAW. This Contract shall be construed in accordance with the Maxims of Equity. 

### Notice
NOTICE. Any notice or communication required or permitted under this Contract shall be sufficiently given if delivered to a verifiable email address or to such other email address as one party may have publicly furnished in writing, or published on a broadcast contract provided by this blockchain for purposes of providing notices of this type. 
  
### Waiver of Contractual Right
WAIVER OF CONTRACTUAL RIGHT. The failure of either party to enforce any provision of this Contract shall not be construed as a waiver or limitation of that party's right to subsequently enforce and compel strict compliance with every provision of this Contract. 

### Arbitrator's Fees to Prevailing Party
ARBITRATOR’S FEES TO PREVAILING PARTY. In any action arising hereunder or any separate action pertaining to the validity of this Agreement, both sides shall pay half the initial cost of arbitration, and the prevailing party shall be awarded reasonable arbitrator's fees and costs.
  
### Construction and Interpretation
CONSTRUCTION AND INTERPRETATION. The rule requiring construction or interpretation against the drafter is waived. The document shall be deemed as if it were drafted by both parties in a mutual effort. 
  
### In Witness Whereof
IN WITNESS WHEREOF, the parties hereto have caused this Agreement to be executed by themselves or their duly authorized representatives as of the date of execution, and authorized as proven by the cryptographic signature on the transaction that invokes this contract.

```

# 번역 정보

* 원문 : https://github.com/eosio/eos/wiki/Tutorial-Hello-World-Contract
* 번역 기준 리비전 : 0c18223ea118597c7b2476118091c4094be6af99
