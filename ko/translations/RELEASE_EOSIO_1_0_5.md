# EOSIO 1.0.5 Release Notes
2018년 6월 16일에 발생한 MainNet 장에를 해결하기 위한 긴급 출시.

지연 트랜잭션이나 유보 트랜잭션의 실행에서 기능 정지(hard failure)가 발생하는 경우에도 블록 생성을 계속하게 함. ([#4158](https://github.com/EOSIO/eos/pull/4158)).

## Pull Request #4158

* heifner : Let's add a method to transaction_receipt for determination of delayed/deferred transactions and unit tests for next release.
* testcode77 : 
If the number of delayed transactions increases, this case is meaningless. My opinion is that the handling of delayed transactions has to change.

## [이슈 #4159](https://github.com/EOSIO/eos/issues/4159) : Throw unexpected fails in apply_block - revisit

[#4158](https://github.com/EOSIO/eos/pull/4158) quick fix needs unittests. Also determine why existing unittests that push blocks to copy of chain did not catch this.

[#4158](https://github.com/EOSIO/eos/pull/4158) bug fix highlights that our determination of delayed/deferred transactions is subtle. transaction_receipt trx variant contains a transaction_id_type for delayed/deferred transactions. Add a method to transaction_receipt which provides clear indication of delayed/deferred. Note that the structure of transaction_receipt can't easily change as it is part of the signed_block.

## 번역 정보

* 원문 : https://github.com/EOSIO/eos/releases/tag/v1.0.5
* 원문 게시일 : 한국표준시 2018년 6월 16일 23시 01분