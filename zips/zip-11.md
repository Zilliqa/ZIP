| ZIP | Title | Status | Type  | Author| Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) |
| --- | ---------------------------- | ------ | ----- | ----------------- | -------------------- | -------------------- |
| 11   | Seed Node Staking Mechanism: Phase I | Draft  | Standards Track | Han Wen Chua <hanwen@zilliqa.com>, <br> Mervin Ho <mervin@zilliqa.com>, <br> Lulu Ren <lulu@zilliqa.com>, <br> Jun Hao Tan <junhao@zilliqa.com>, <br> Antonio Nunez <antonio@zilliqa.com>,  and <br> Amrit Kummer <amrit@zilliqa.com> | 2020-08-17| 2020-08-17|


## Abstract

ZIP-11 is the Phase I extension of the seed node staking proposal as presented in [ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md). This new proposal introduces a non-custodial mechanism to stake tokens with a seed node operator via a Scilla contract. Non-custodial here meaning that any tokens that need to be staked can now be deposited directly in the contract and therefore need not go through any intermediary entity acting as a custodian. ZIL-11 also introduces some key changes in the staking parameters such as lifting the cap on the total stake deposit (a consequence of the tokenomic changes as proposed in [ZIP-9](https://github.com/Zilliqa/ZIP/blob/zip-9/zips/zip-9.md)).


## Backward Compatibility

The staking mechanism is intended to work alongside the existing core protocol and should be fully backward compatible with all its components, i.e., no change in behavior should be observable on the mainnet operation with this mechanism in place.

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

