|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 1  | Zilliqa Internal Transactions | Draft | Standards Track  | Haichuan Liu <haichuan@gmail.com> <br> Xiaohuo Ren <lulu@zilliqa.com>| 2019-06-23 | 2020-01-30

## Abstract

This ZIP details the standard of how internal transactions are presented externally by the Zilliqa blockchain.

## Motivation

As more and more contract being deployed in Zilliqa blockchain, in some cases, precise tracking of inter-contract transitions and balance transferring is needed, such as to know the rewardees of a bounty contract and the amount they are rewarded. Since the transaction sent to blockchain only indicates which contract and what transition to be invoked, it's unable to tell what exactly happened during the contract execution, which may cause great inconvenience for contract developers and users.

Hence, this ZIP proposes that those internal can be recorded in transaction receipt to help users to track all the information of a contract transaction.

## Specification

Please refer to [`GetTransaction`](https://apidocs.zilliqa.com/#gettransaction) for more details of the transaction receipt specifications.

The internal transaction records will be added as a new field named as `transitions` in the JSON message of API result under the `receipt`. Each entry in `transitions` field consists the following aspects:

|      Field    |                          Description                            |
| ------------- | --------------------------------------------------------------- |
| **  addr   ** | address of the contract emitted this transition                 |
| **  depth  ** | the depth of current transition if happen to invoke a tree call |
| **  msg    ** | the message emitted by scilla interpreter for this transition   |

- The `msg` field is fetched from the `"message"` field of the Scilla interpreter output, for more detail, please refer to [`here`](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output). Do note that this may change if Scilla upgraded its specification.

## Examples

```
"transitions": [
                {
                    "addr": "0x8b46edcfcdb5613da479805f9a943b4a75e544a5",
                    "depth": 0,
                    "msg": {
                        "_amount": "0",
                        "_recipient": "0x7825250c716f71e63c3819c029305ba52680c998",
                        "_tag": "decreaseAllowance",
                        "params": [
                            {
                                "type": "ByStr20",
                                "value": "0xb2e51878722d8b6d2c0f97e995a7276d64c1618b",
                                "vname": "spender"
                            },
                            {
                                "type": "Uint128",
                                "value": "10000",
                                "vname": "value"
                            },
                            {
                                "type": "ByStr20",
                                "value": "0x9bfec715a6bd658fcb62b0f8cc9bfa2ade71434a",
                                "vname": "initiator"
                            }
                        ]
                    }
                }
            ]
```

## Rationale

### Two possible solutions:

To solve the problem, actually two possible solutions were raised during the discussion. The first one has been discribed above. Another one is to let seed node to run all the transactions happened in the previous epoch based on the states in the last epoch, then aggregates all the output emitted by the local Scilla interpreter and provide to the external interface. The following chart describes the mechanisms, pros and cons of two approaches:

|   solution    | mechanism | pros | cons |
| ------------- | --------- | ---- | ---- |
| **receipt**   | records all the internal transitions in transaction receipt | - simple <br> - no extra computation | - old transactions not supported |
| **seed node** | seed node replay all transactions| - consensus independent, save bandwidth <br> - backward compatible | - difficult to implement <br> - heavy computation needed, may exceed epoch time since it burdens the computation over multiple shards|

Considering all the aspects above, the receipt approach is chosen for current adoption. However we will still be watching out for the potential of the second approach can bring along the way.

## Backwards Compatibility

Due to transaction receipt hashing, all the receipts have been generated before are immutable, thus only receipt generated after the [`release v6.1.0`](https://github.com/Zilliqa/Zilliqa/releases/tag/v6.1.0) can enjoy this feature.

## Reference
- [`Zilliqa API docs for GetTransaction`](https://apidocs.zilliqa.com/#gettransaction)
- [`Scilla doc for interpreter output`](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
