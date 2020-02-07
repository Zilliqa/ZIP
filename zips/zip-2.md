|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 1  | Zilliqa Internal Transactions | Draft | Standards Track  | Haichuan Liu <haichuan@zilliqa.com> <br> Xiaohuo Ren <lulu@zilliqa.com>| 2020-01-15 | 2020-02-05

## Abstract

This ZIP details the standard of how internal transactions are presented externally by the Zilliqa blockchain.

## Motivation

As more and more contracts are deployed on the Zilliqa blockchain, there becomes a need for a precise way to track inter-contract transitions and balance transfers in some cases (e.g., the rewardees and their corresponding reward amounts in the case of a bounty contract). Since a transaction sent to the blockchain only indicates which contract and what transition to be invoked, there is no apparent way to tell what exactly happened during the contract execution, which causes inconveniences to contract developers and users.

Hence, this ZIP proposes a way for users to track all the information in a contract transaction by recording all internal transactions in the contract transaction's receipt.

## Specification

Please refer to [`GetTransaction`](https://apidocs.zilliqa.com/#gettransaction) for more details of the transaction receipt specifications.

The `GetTransaction` API call returns a JSON message that includes a `receipt` field. Under this `receipt`, the internal transaction records are added as individual entries under a new field named `transitions`. Each entry in `transitions` further consists of the following fields:

|      Field    |                          Description                            |
| ------------- | --------------------------------------------------------------- |
|     `addr`    | Address of the contract that emitted this transition            |
|     `depth`   | Depth of current transition if a tree call is invoked           |
|     `msg`     | Message emitted by the Scilla interpreter for this transition   |

The `msg` field is fetched from the `message` field of the Scilla interpreter output, for more detail, please refer to the [Scilla specification](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output).

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

There are at least two possible solutions to the problem of tracking internal transactions.

The first one is the one put forward in this proposal, i.e., the recording of those transactions in the contract transaction's receipt.

The alternative solution is to require seed nodes to re-execute confirmed transactions in each epoch, and based on the states from the last epoch. The output emitted by the seed node's Scilla interpreter are then aggregated and made accessible through an API call.

The following table summarizes the mechanism and pros and cons of each approach.

|   Solution    | Mechanism                                                   | Pros | Cons |
| ------------- | ----------------------------------------------------------- | ---- | ---- |
| Receipt       | Records all the internal transitions in transaction receipt | - Simple <br> - No extra computation | - Old transactions are not supported |
| Seed node     | Seed node replays all transactions                          | - Consensus-independent, saving bandwidth <br> - Backward compatible | - Difficult to implement <br> - Computationally heavy and may exceed epoch time since it must be performed over all the shards' transactions |

Considering all the aspects above, the receipt approach is chosen for current adoption.

## Backward Compatibility

Due to transaction receipt hashing, all the receipts that have been generated before this ZIP is implemented are immutable. Thus, only receipts generated after implementation can benefit from this feature, making this backward incompatible.

## References
- [Zilliqa API documentation for GetTransaction](https://apidocs.zilliqa.com/#gettransaction)
- [Scilla documentation for interpreter output](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
