|  ZIP | Title | Status| Type | Author                                                                   | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--|--------------------------------------------------------------------------| -- | -- |
| 24  | On-Chain Rewards | Approved | Standards Track  | Richard Watts <richard@zilliqa.com>, Zoltan Fazekas <zoltan@zilliqa.com> | 2023-10-17 | 2023-12-12

## Abstract

This ZIP describes how the governance proposal in GP008 (dynamic staking rewards) is implemented, in order that miners and others can write scripts to read off the current reward structure.

## Motivation

Zilliqa rewards are generated on a per-epoch basis; however, as our
network has become more efficient, blocks and therefore epochs have
been generated more rapidly and the number of rewards paid out has
increased.

We need to be able to tune the staking reward level so as not to
exhaust the supply of ZIL.

Governance proposal GP008 was passed on 2023-10-14 to address this,
and this ZIP describes its implementation.

We also describe the implementation of some tunables surrounding the
implementation of ZIP-23 - in particular, the multipliers for fast and
slow miners depend on the structure of the DS committee; we will tune
these in order to achieve the reward distribution in ZIP-23.

## Specification

There is a contract at a well-known address on the blockchain, which
will be advertised in the `constants.xml` available for download with
the key:

```
<transactions>
<REWARD_CONTROL_CONTRACT_ADDRESS>(address)</REWARD_CONTROL_CONTRACT_ADDRESS>
</transactions>
```

If this contract does not exist, or is not a contract, or is a
contract which does not contain the fields specified below, the chain
will use the existing values and the constants in ZIP-23.

If the contract does exist, it will be a Scilla contract containing the following fields:

```
(* Reward per DS in Qa *)
field coinbase_reward_per_ds: Uint128

(* Reward percentages, scaled by percent_precision - 100 means % * 100 *)
field base_reward_in_percent : Uint32
field lookup_reward_in_percent : Uint32
field node_reward_in_percent : Uint32
field percent_precision : Uint32

(* The denominator here is a fixed 1000, so 1.668 is represented as 1668*)
field reward_each_mul_in_millis : Uint32
field base_reward_mul_in_millis : Uint32
```

You can read off the reward to be given at the end of the current epoch from these fields.

## Implementation

Whilst you should not rely on it, the reward control contract that is expected to be used is in the `zilliqa-developer` repository, at
`contracts/reward_control/contracts/scilla/rewards_params.scilla`

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
