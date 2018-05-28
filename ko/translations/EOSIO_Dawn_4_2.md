# EOSIO Dawn 4.2 출시

EOSIO 버전 1.0이 바로 코 앞에 있는 상황에서, 오늘 우리는 EOSIO 소프트웨어의 Dawn 4.2 릴리즈를 태그했습니다. 몇몇 작은 기능이 코드에 추가되었지만, 아래에서 볼 수 있는 내용과 같이, 우리는 버그 픽스와 안정성 개선에 집중해오고 있습니다.

## 중요한 변경 사항

**EOSIO.SYSTEM 자금 분리**

시스템 자금 할당이 확실한지 확인하기 위해서, 시스템 자금을 다음과 같이 분리하기로 결정했습니다.

*    모든 램 구매와 판매에 1%의 수수료를 구현함
*    사용자가 지불하는 모든 램 거래 수수료는 `eosio.ramfee`로 지불됨
*    램 판매의 모든 수익은 `eosio.ram`으로부터 보냄
*    지분참여한(staked) 토큰 전부는 `eosio.stake`로 보냄
*    지분참여가 해제된(unstaked) 토큰 전부는 `eosio.stake`로부터 보냄
*    계정 이름 경매 수익의 전부는 `eosio.names`로 보냄
*    할당되지 않은 인플레이션(통화 증가량)은 `eosio.saving`로 보냄
*    프로듀서 블록 생성 보상(producer block pay)은 `eosio.bpay`로 보냄
*    프로듀서 투표 획득 보상(producer vote pay)은 `eosio.vpay`로 보냄

자세한 사항은 `contracts/eosio.system/producer_pay.cpp`를 확인하세요.

**Chain Plugin**

디스크에 저장되는 데이터의 몇몇에 대해 그리고 노드를 동작시키는데 사용하는 사용자에게 노출된 옵션 몇몇에 대해 약간의 혼란이 커뮤니티에 있다는 것을 지켜봤습니다. 그래서 다음 업데이트를 통해서 보다 분명하게 설명하려고 합니다.

우리는 "shared_mem" 디렉토리를 "state"라고 이름을 바꾸었고, 플러그인 옵션 몇 개를 변경했습니다.

*    "--block-log-dir" 옵션을 "--blocks-dir"로 바꿈
*    "--checkpoint" 의 단축 옵션이었던 "-c" 를 제거함
*    "--shared-memory-size-mb" 옵션을  "--chain-state-db-size-mb"로 바꿈
*    "--reversible-blocks-db-size-mb" 옵션을 추가해서 가역(reversible) 블록 DB의 초기값을 변경할 수 있도록 함
*    "--resync-blockchain" 옵션을 "--delete-all-blocks"로 바꾸고, 이 옵션의 사용을 지양하도록 함
*    "--contracts-console" 옵션은 컨트랙트 출력을 콘솔에 나오도록 하고 config.ini에서 설정 가능함
*    "--hard-replay-blockchain" 옵션을 추가해서 블록 로그와 데이터를 백업 디렉토리에 옮길 수 있게 하고, 새로운 "blocks.log"를 만들어서 마치 "--replay-blockchain" 옵션이 사용된 것처럼 이어서 진행하게 함
*    "--force-all-checks" 옵션을 추가해서 수신한 트랜잭션, 인라인 액션, 컨트랙트 내부에서 생성된 트랜잭션에 대해 강제로 권한 승인 여부를 검증하게 함
*    "--fix-reversible-blocks" 옵션을 추가해서 nodeos로 하여금 "가역적인" 블록 데이터베이스로부터 복구를 시도하고 이어서 바로 종료(exit)하게 함
*    "--hard-replay-blockchain" 옵션의 동작을 업데이트해서, "가역적인" 블록 데이터베이스로부터, 이 데이터베이스가 지저분한 상태(dirty state)로 남아있다고 해도, 최대한 많은 가역 블록을 복구하고 replay하도록 시도함
*    "--genesis-json" 옵션을 추가해서 genesis state를 읽어올 파일을 명시함
*    "--genesis-timestamp" 옵션을 추가해서 Genesis State 파일의 초기 타임스탬프를 덮어씀
*    "--print-genesis-json" 옵션을 추가해서 blocks.log로부터 genesis state 정보를 JSON 형태로 추출해서 콘솔에 출력함
*    "--extract-genesis-json" 옵션을 추가해서 blocks.log로부터 genesis state 정보를 JSON 형태로 추출한 후 명시한 파일에 저장함

**Producer Plugin**

우리는 커맨드 라인 옵션 하나 (--max-irreversible-block-age)를 추가했고, 이 옵션은 마지막 비가역 블록(last irreversible block) 시간이 N초보다 오래되면 노드로 하여금 자동적으로 블록 생산을 중단하게 만듭니다. LIB 시간이 N초보다 오래되는 상황은 체인이 계속 생성되기에는 블록을 승인하는 프로듀서가 충분하지 않다는 점을 암시합니다. 이 옵션은, 충분한 수의 프로듀서가 네트워크에 다시 합류하게 되었을 때 블록 생성을 재개하기 위해 합리화가 필요한 과도한 수의 블럭을 생성하지 못하게 막습니다.

우리는 노드가 블록 생성을 필요에 따라 잠깐 멈출 수 있는 기능을 추가했습니다. 이 기능의 목적은, 나쁜 상황이 발생하는 동안 체인 replay/resync 등의 시간 소모가 생기는 가변성 없이 프로듀서가 체인 생산을 잠시멈춤/재개하는 조정이 가능하도록 하기 위해서입니다.

참고로 다음과 같은 RPC API가 있습니다.
POST /v1/producer/pause - 블록 생산 일시 정지
POST /v1/producer/resume - 일시 정지된 블록 생산을 재개\
POST /v1/producer/paused - 현재 노드가 일시 정지된 상태인지 아닌지에 따라 true/false를 돌려줌

**Genesis JSON 파일은 더이상 기본으로 생성되지 않음**

`genesis.json`은 더이상 config 디렉토리에 있을 것이라고 예상되지 않고, 자동으로 생성되지 않을 것입니다. 아래는 앞으로 실행해야 할 명령어입니다.

`nodeos --extract-genesis-json <genesis filename>`

파일 내용을 변경하고, 타임스탬프를 바꾸고, 초기 프로듀서 키와 보통 변경하는 다른 파라미터를 바꾸세요. 과거에 실행했던 것처럼 파일을 공유해서 사용하세요.

`nodeos --delete-all-blocks --genesis-json <genesis filename>` 명령으로 (모든 노드에서) 새로운 블록체인을 초기화합니다.

이어지는 론칭 과정에서, `--genesis-json` 파일을 명시하지 마십시오. 그러면 nodeos는 블록 로그로부터 genesis state를 가져올 것입니다. 그리고 물론 `--delete-all-block` 옵션도 사용하지 마세요.

nodeos가 깨끗하게 재시작하지 못하면, `--replay-blockchain`를 사용하시고, 만약 그래도 제대로 동작하지 않는다면 `--hard-replay-blockchain`를 사용하세요.

resync 옵션은 실제로 동작하는 내용을 반영하기 위해서 `--delete-all-blocks`로 변경되었습니다.

**BIOS Boot Process 튜토리얼 업데이트**

우리는 프로듀서가 보상을 청구하는 부분, 위임(프록시) 투표, `eosio` 계정 만료(resign), 프로듀서가 `eosio.msig` 컨트랙트를 이용해서 `eosio.system` 컨트랙트를 교체하는 과정과 몇몇 버그 수정 등을 포함해서 BIOS Boot 튜토리얼 내용을 개선했습니다. 튜토리얼에서 설명하는 파일들을 "programs" 디렉토리에서 새로운 "tutorials" 디렉토리로 옮겼다는 점을 참고해주세요.

**다른 수정 사항**

지난주 "Quality Name Distribution & Namespaces" ([#3189](https://github.com/eoseoul/docs/blob/master/ko/translations/EOSIO_Dawn_4_1.md#%EA%B9%83%ED%97%99-%EC%9D%B4%EC%8A%88-3189)) 제안에서 언급한 내용과 같이, 계정 이름 경매가 구현되었고, 네트워크가 언락된 (15%의 투표) 이후 2주 동안 활성화되지 않을 것입니다.

프로듀서는 앞으로 RAM 크기를 증가시킬 수만 있고 절대로 줄일 수 없습니다. 블록 프로듀서에 의한 램 시장의 잠재적인 조작을 방지하기 위해서 그렇게 구현했습니다.

Boost 라이브러리 버전을 1.66에서 1.67로 업데이트했습니다.

"data/shared_mem" 디렉토리가 "data/state"로 변경되었습니다.

config.ini의 옵션 중 "block-log-dir"는 "blocks-dir로 변경되었습니다.

새로운 활성 프로듀서 스케줄을 제청한다(propose)는 함수의 의도를 보다 정확히 반영하기 위해서, "set_active_producers" 기본 메소드를 "set_proposed_producers"로 변경했습니다.

지난주에 토큰 심볼 변경에 따라서, Dockerfile에 코어 토큰 심볼을 명시할 수 있는 기능을 추가했습니다. 우리는 또한 Docker 빌더 이미지로 Ubuntu 18.04 LTS로 바꿨습니다.

체인을 초기화할 때 설정한 genesis state의 SHA256 다이제스트가 이제 체인 ID입니다. `--genesis-json`을 사용해서 덮어쓰지 않으면, nodeos는 소프트웨어의 genesis state 기본값을 사용할 것입니다. genesis state 기본값의 초기 root key는 EOSIO_ROOT_KEY CMake 옵션을 이용해서 변경할 수 있습니다.

블록을 생성하는 프로듀서가 다음 프로듀서로 생성 순서를 넘길 때(handoff), 특히 프로듀서 사이의 네트워크 레이턴시가 높은 경우, 작은 포크(mini fork)가 예상치 않게 높은 빈도로 발생한다는 사실을 발견했고, 수정했습니다.

블록 프로듀서는 시스템 컨트랙트에 새로 추가된 "setparams" 액션을 이용해서 블록체인 파라미터를 이제 변경할 수 있게 되었습니다.

몇몇 테스트넷에서 프로듀서 투표로 비활성화되는 유예 시간이 너무 빡빡하다는 점에 주목했습니다. 유예 시간이 대략 4분에서 7시간으로 변경되었습니다. 비활성화와 관련된 몇몇 코너 케이스 역시 수정되었습니다.

우리는 명시된 필터 규칙에 맞는 액션만 추적하도록 history 플러그인을 수정해서 history 플러그인의 메모리 사용량을 크게 줄였습니다. 모든 액션을 추적하면 shared_mem을 채우기 쉽기 때문에, 이제 모든 액션을 추적하는 방법은 없습니다.

우리는 비정상적이고 다양한 ABI 정의를 이용한 무한 재귀 버그를 발견했고, 고쳤습니다.

우리는 블록 프로듀서가 블록 생성 스케줄에서 적절하지 않게 제외되지 않도록 프로듀서 비활성화 로직을 변경했습니다. 우리는 이제 각각의 프로듀서 정보인 "time_became_active" 값을, 처음으로 블록 프로듀서로 선출되는 시점에서만 업데이트하지 않고, 프로듀서로 선출될 때마다 업데이트합니다. 만약 처음으로 선출되었거나 마지막으로 선출된지 하루 이상이 지났다면, 우리는 "last_produced_block_time"을 체크하지 않으며 비활성화시키지 않습니다. 그렇게 하지 않으면 로직은 옛날과 같이 동작할 것입니다. 이런 방법으로, 투표에서 밀려난 탓에 블록을 생산하지 못한 프로듀서를 불리하게 만들지 않습니다.

## 패트로니오스 - Patroneos

우리는 지난주에 패트로니오스를 소개했고, 이제 EOSIO 깃헙 저장소 https://github.com/EOSIO/patroneos 를 통해서 커뮤니티에 제공합니다.

상기시키는 차원에서 다시 말씀드리자면, 패트로니오스는 기초적인 서비스 거부 공격(Denial of Service) 경로를 방어하기 위해 설계된 보호 계층을 EOSIO 노드에 제공합니다.

## 깃헙에 올라온 Dawn 4.2

EOSIO Dawn 4.2가 깃헙에 올라와 있습니다.(https://github.com/EOSIO/eos/releases/tag/dawn-v4.2.0) 이를 이용해서 개발자들과 블록 프로듀서 출마자들은 애플리케이션과 네트워크를 계속 테스트할 수 있습니다..

## 커뮤니티 지원

상기하는 차원에서 다시 말씀드리지만, 우리는 커뮤니티로부터 피드백을 듣는 게 참 좋고, EOSIO를 성공적으로 동작시키도록 커뮤니티를 돕는 데 헌신하고 있습니다. 우리의 EOSIO 스택 익스체인지 (https://eosio.stackexchange.com/) Beta는 잘 성장하고 있고, 이 소프트웨어를 사용하는 방법에 대한 질문을 스택 익스체인지에 올려주시길 장려합니다. 아래 정보를 포함시켜서 깃헙에서 발견할 지도 모르는 잠재적인 버그를 계속 리포트해주시길 바랍니다.

* Testnet: (옮긴이 주: 이슈를 발견한 테스트넷이 어디인지)
* EOSIO git version: `nodeos -v output` 결과 문자열
* config.ini: 사용하고 있는 `config.ini` 파일을 첨부해주세요
* genesis.json: 사용하고 있는 `genesis.json` 파일을 첨부해주세요
* Command line: 사용한 `nodeos` 전체 커맨드 명령어
* Console output: 확인하고 있는 에러와 관련된 콘솔 디버그 출력값 (짧으면 아예 이슈 설명에 붙여넣어도 좋습니다)

## 추후 릴리즈

우리는 6월 1일에 EOSIO 1.0을 예정하고 있습니다. 깃헙의 버전 1.0 마일스톤(https://github.com/EOSIO/eos/milestone/10) 을 지켜보시면 다음 몇 주간 우리가 집중해서 작업하고 있는 이슈들을 따라잡으실 수 있습니다.


## 번역 정보

* 원문 : https://github.com/EOSIO/eos/releases/tag/dawn-v4.2.0
* 원문 첫 게시 시점 : 2018년 5월 26일 9시 44분 KST
