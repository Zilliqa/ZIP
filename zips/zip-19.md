| ZIP | Title | Status | Type  | Author| Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) | 
| --- | ---------------------------- | ------ | ----- |----------------- | -------------------- |-------------------- | 
| 19   | Seed Node Staking Mechanism: Phase 1.1 | Draft  | Standards Track | Te Ye Yeo <teye@zilliqa.com>, <br> Jun Hao Tan <junhao@zilliqa.com>, Lulu Ren <lulu@zilliqa.com>, <br> Amrit Kummer <amrit@zilliqa.com> | 2021-03-25 | 2021-04-05 |

# Table of Content

- [Abstract](#abstract)
- [Background](#background)
  * [Limitation around smart contract wallets](#limitation-around-smart-contract-wallets)
  * [Incomplete removal of empty maps in contract states](#incomplete-removal-of-empty-maps-in-contract-states)
  * [Compatibility issue with upcoming Scilla version upgrade](#compatibility-issue-with-upcoming-scilla-version-upgrade)
- [Design changes for Phase 1.1](#design-changes-for-phase-11)
  * [Transfer of stake deposit between accounts](#transfer-of-stake-deposit-between-accounts)
  * [Proper deletion of empty map entries](#proper-deletion-of-empty-map-entries)
  * [Incomplete removal of empty maps in contract states](#incomplete-removal-of-empty-maps-in-contract-states)
- [Contract Specification and Implementation](#contract-specification-and-implementation)
- [Limitations and Future Work](#limitations-and-future-work)
- [Backward Compatibility](#backward-compatibility)

# Abstract

ZIP-19 presents the Phase 1.1 extension of the seed node staking proposal as
presented in [ZIP-11](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md).

This new proposal introduces a new mechanism to transfer stake from one account to another account. 
In addition, it also fixes an empty map deletion bug in Phase 1 and resolve an compatibility with upcoming Scilla version `0.10.0` upgrade.  

# Background

We strongly recommend readers to go through
[ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md),
[ZIP-9](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-9.md) and [ZIP-11](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-11.md)
for background on Zilliqa Seed Node Staking Program.

The Staking phase 1 have been a success and many Zilliqa wallets provider such as [Atomic](https://atomicwallet.io/staking), 
[Frontier](https://frontierwallet.com/), [Moonlet](https://moonlet.io/#staking), [Zillet](https://zillet.io/) 
and [Zillion](https://stake.zilliqa.com/) have successfully integrated Zilliqa Seed Node staking.

However, the team have receive feedbacks on how to further improve the Zilliqa non-custodial seed node staking program.

## Limitation around smart contract wallets

The current staking contracts for phase 1 is not versatile enough to support smart contract wallets. For smart contract wallets, 
if the wallet provider need to "upgrade" or add new features to the smart contract, a new deployment of the smart contract and follow 
by a transfer of assets to the new smart contract is required. Under Staking phase 1, this means that the delegator has to withdraw all his stakes, 
wait for at least 24,000 blocks for stake deposit to be unbonded before $ZIL can be transferred to the new smart contract wallet. 

## Incomplete removal of empty maps in contract states

Maps entries that are not required are deleted from the contract state. However, due to a improper deletion in phase 1, rather than map entries being deleted, 
the map entries are cleared instead. This cause the contract data size to be bloated unneccessary.

### Example:
Expected map content after withdrawal of stake deposit is completed

Current:
```
  "0x....": {}, <-- unneccessary map entry
  "0x....": {
    "1067644": "45118000000000000"
  }

```

Expected:
```
  "0x....": {
    "1067644": "45118000000000000"
  }
```

The maps affected are 
- `buff_deposit_deleg`
- `deleg_stake_per_cycle`
- `deposit_amt_deleg`
- `direct_deposit_deleg`
- `last_buf_deposit_cycle_deleg`
- `last_withdraw_cycle_deleg`
- `withdrawal_pending`
- `ssn_deleg_amt`

## Compatibility issue with upcoming Scilla version upgrade

The upcoming Scilla version `0.10.0` contains a bug fix for an issue known as `disambiguation bug`. This bug fix is neccessary to support future Scilla 
features such as remote state read and Scilla external library. After the fix, calling a contract via our JSON RPC APIs and passing a custom user-defined ADT as transition parameter will requires contract addres to be included. Hence, the staking `proxy` contract is no longer compatibile with the `ssnlist` contract starting from Scilla version `0.10.0`.

# Design changes for phase 1.1

## Transfer of stake deposit between accounts

A new feature will be added to allow transferring of entire stake deposit, rewards and pending withdrawal across all SSNs to a new address. Such a transfer will not unstake the existing stake deposit and transfer will be immediately executed upon confirmation of the transfer. There is no penalty incurred for this transfer and 
there is no restriction on the number of transfers.

### Scenario 1: Transferring to an address which does not have any active staking activity

All stake deposit and rewards will be transfer ot the new address

### Scenario 2: Transferring to an address which currently have 1 or more stake delegation or pending stake withdrawal

All stake deposit and rewards will be transfer ot the new address. If there is any overlap with the existing stake delegation, rewartds or pending withdrawal, the
transferred value will be added to the exisiting stake on the new address.

### Mechanism 

The transfer will adopt a two step process. First the transferer will need to initiate a request to transfer to another address. The receiptant will then need to 
confirm the transfer. Upon confirmation, the transfer will be executed. 

3 new transitions in both `proxy` and `ssnlist` will be added to support this new feature

| transition | Comments |
| ---------- | -------- | 
| `RequestDelegatorSwap` | To initiate the request to transfer all existing stake deposit, rewards and pending withdrawals |
| `ConfirmDelegatorSwap` | To execute the transfer |
| `CancelDelegatorSwap` | to cancel the transfer request |

### Caveat

This mechanism does not support partial transfer.

## Proper deletion of empty map entries

In order to remove the empty map entries properly, Scilla builtin `remove` should be used instead of `delete`. Upon removal, the updated sub-map should be 
re-assigned to the mutable variable again.

## Removal of custom ADT at `AssignStakeRewards` transition

To resolves the `disambiguation bug`, the custom ADT `SsnRewardShare` in `AssignStakeReward` transition parameter 
will be modify to `List (Pair ByStr20 Uint128))` in both `proxy` and `ssnlist` contracts.

# Contract Specification and Implementation

More details on the contracts can be found in the [staking contract
repository](https://github.com/Zilliqa/staking-contract/blob/dev/contracts/README.md).

For direct access:

* The specification for the different contracts needed for Phase 1.1 can be 
found [here (TBA)](https://github.com/Zilliqa/staking-contract/blob/dev/contracts/README.md). 

* The implementation of the contracts can be found 
[here (TBA)](https://github.com/Zilliqa/staking-contract/tree/dev/contracts). 

# Limitations and Future Work

ZIP-19 aims to make minor functional improvements and fixes to the non custodial staking contract. 
Other major contracts enhancements will be left as a future work for future phases of staking. 

# Backward Compatibility

The non-custodial staking will be implemented as a new contract and the older
contract will be deprecated. Contract data will be migrated over to the new contract, 
through a migration exercise that is likely to last at least a week. The old contract will
be freeze permanently. 

## Copyright Waiver

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
