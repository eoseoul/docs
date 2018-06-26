# EOSIO v1.0.4 Release Notes

## Usability Updates

There were several improvements to error messages, help text, command line options, and outputs in this release. The `--unlock-timeout` option was removed due to some potential confusion ([#4139](https://github.com/EOSIO/eos/pull/4139)). We added some additional security and warning messages ([#4083](https://github.com/EOSIO/eos/pull/4083)) and the ability to squelch debugging output over http ([#4103](https://github.com/EOSIO/eos/pull/4103)).

Previously, as a convenience for tutorials, there was a default key created by keosd meant only for developing and testing purposes. This was confusing some users who inappropriately used that key for their own transactions on the main-net. The system no longer generates a default key and a new feature has been added to the Wallet API plugin to remove an existing key from a wallet ([#4107](https://github.com/EOSIO/eos/pull/4107)).

The Chain API plugin has been modified to now also provide unstaking information of accounts. This accompanies cleos usability improvements to the `get account` subcommand which now will display the core tokens an account is currently unstaking ([#4063](https://github.com/EOSIO/eos/pull/4063)) as well as their liquid core token balance ([#4082](https://github.com/EOSIO/eos/pull/4082)). The changes to the Chain API should not break older versions of clients like cleos and likely other existing wallets. Furthermore, v1.0.4 cleos should still be compatible with API nodes running on version v1.0.3.

### System Contract
There were a couple of changes to the system contract. RAM fees are now rounded up, so effectively everyone pays a non-zero fee for buying RAM, no matter how small the purchase ([#4051](https://github.com/EOSIO/eos/issues/4051)). And a bug which delayed the activation of name bidding has been fixed ([#4106](https://github.com/EOSIO/eos/pull/4106)). These changes to the system contract are compatible with the existing table data so upgrading should be straightforward. We have included a [step-by-step guide](https://github.com/EOSIO/eos/wiki/Upgrading-the-system-contract) on our Wiki to help block producers with the process of upgrading the system contract.

### New Blacklist Feature
There is a new subjective blacklist feature, `key-blacklist`, which allows producers to add public keys to their config.ini which are disallowed from being used in the authorities of account permissions. Any transaction that tries to add/modify a permission authority (for example through `eosio::newaccount` or `eosio::updateauth`) to contain a blacklisted public key is subjectively rejected by the producer ([#4110](https://github.com/EOSIO/eos/pull/4110)).

## Security Updates

We continue to update other areas based on reports through the bug bounty program.
## Consensus Changes from v1.0.3

None

## 번역 정보

* 원문 : https://github.com/EOSIO/eos/releases/tag/v1.0.4
* 원문 게시일 : 한국표준시 2018년 6월 16일 오전 7시 59분