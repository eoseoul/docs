# EOSIO 1.0.5 Release Notes
2018년 6월 16일에 발생한 MainNet 장에를 해결하기 위한 긴급 출시.

지연 트랜잭션이나 유보 트랜잭션의 실행에서 기능 정지(hard failure)가 발생하는 경우에도 블록 생성을 계속하게 함. ([#4158](https://github.com/EOSIO/eos/pull/4158)).

## Pull Request #4158

* heifner : transaction_receipt에 메소드를 하나 추가해서 지연/유보 트랜잭션을 결정하게 하고, 다음 버전에서 유닛 테스트를 추가하겠습니다.
* testcode77 : 지연된(delayed) 트랜잭션의 수가 증가하면, 이 경우가 무의미해집니다. 제 생각으로는 지연 트랜잭션을 다루는 방법이 변해야 합니다.

## [이슈 #4159](https://github.com/EOSIO/eos/issues/4159) : Throw unexpected fails in apply_block - revisit

[#4158](https://github.com/EOSIO/eos/pull/4158) 퀵 픽스에 따른 유닛 테스트가 필요함. 또한 체인에 블록을 푸시하는 현재의 유닛 테스트가 이 경우를 잡아내지 못한 이유를 찾아야 함.

[#4158](https://github.com/EOSIO/eos/pull/4158) 버그 픽스는 우리가 지연/유보 트랜잭션을 결정하는 방식이 미묘하다는 점을 강조하고 있음. transaction_receipt trx variant는 지연/유보 트랜잭션에 대해 transaction_id_type를 가짐. transaction_receipt에 메소드를 추가해서 명확하게 유보/지연 트랜잭션임을 드러내게 함. transaction_receipt의 구조는 그것이 signed_block의 일부이기 때문에 쉽게 변할 수 없다는 점을 주의해야 함.

## 번역 정보

* 원문 : https://github.com/EOSIO/eos/releases/tag/v1.0.5
* 원문 게시일 : 한국표준시 2018년 6월 16일 23시 01분