| ZIP | Title | Status | Type  | Author| Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) | 
| --- | ---------------------------- | ------ | ----- |----------------- | -------------------- |-------------------- | 
| 19   | Seed Node Staking Mechanism: Phase 1.1 | Implemented  | Standards Track | Te Ye Yeo <teye@zilliqa.com>, <br> Jun Hao Tan <junhao@zilliqa.com>, Lulu Ren <lulu@zilliqa.com>, <br> Amrit Kummer <amrit@zilliqa.com> | 2021-03-25 | 2021-05-11 |

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
presented in [ZIP-11](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-11.md).

This new proposal introduces a mechanism to transfer stake from one account to another. 
Additionally, it also fixes a bug related to incomplete deletion of empty maps and resolves compatibility issues with the upcoming Scilla version `0.10.0` upgrade.

The Zilliqa team will facilitate the migration of contract states from Phase 1 to Phase 1.1 via remote state read, which will be part of Zilliqa v8.0.0 release.

# Background

We strongly recommend readers to go through
[ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md),
[ZIP-9](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-9.md) and [ZIP-11](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-11.md)
for background on Zilliqa Seed Node Staking Program.

Seed Node Staking (Phase 1) has been a success and many Zilliqa wallet providers such as [Atomic](https://atomicwallet.io/staking), 
[Frontier](https://frontierwallet.com/), [Moonlet](https://moonlet.io/#staking), [Zillet](https://zillet.io/) 
and [Zillion](https://stake.zilliqa.com/) have successfully integrated Zilliqa Seed Node staking.

However, the team has received some feedback on how to further improve the user experience. This proposal aims to cater to one such feedback.
## Limitation around smart contract wallets

The current staking contracts for Phase 1 are not versatile enough to support smart contract wallets. For smart contract wallets, 
if the wallet provider needs to "upgrade" or add new features to the smart contract, a new deployment of the smart contract, followed 
by a transfer of assets to the new smart contract is required. With Staking Phase 1, this means that the delegator has to withdraw all his stakes, 
wait for at least 24,000 blocks for stake deposit to be unbonded before $ZIL can be transferred to the new smart contract wallet. 

## Incomplete removal of empty maps in contract states

Maps entries that are not required are deleted from the contract state. However, due to an incomplete deletion in Phase 1, rather than map entries being deleted, 
the map entries are cleared instead. This causes the contract data size to be get unnecessarily bloated.

### Example:
Expected map content after withdrawal of stake deposit is completed

Current:
```
  "0x....": {}, <-- unnecessary map entry
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
- `direct_deposit_deleg`
- `deleg_stake_per_cycle`
- `deposit_amt_deleg`
- `last_buf_deposit_cycle_deleg`
- `last_withdraw_cycle_deleg`
- `withdrawal_pending`

## Compatibility issue with upcoming Scilla version upgrade

The upcoming Scilla version `0.10.0` contains a bug-fix for an issue known as `disambiguation bug`. This bug fix is necessary to support future Scilla 
features such as remote state read and Scilla external library. Starting from Scilla version `0.10.0`, calling a contract via our JSON RPC APIs and passing a custom user-defined ADT as transition parameter will require the contract address to be included. Due to this change, the staking `proxy` contract will no longer compatible with the `ssnlist` contract starting from Scilla version `0.10.0`.

# Design changes for phase 1.1

## Transfer of stake deposit between accounts

A new feature will be added to allow transferring of the entire stake deposit, rewards and pending withdrawals across all SSNs to a new address. Such a transfer will not require the user to go through an unbonding period and instead the transfer of stake and other relevant state will be immediately executed upon confirmation of the transfer. No penalty will be incurred for this transfer and 
there will be no restriction on the number of transfers.

### Scenario 1: Transferring to an address which does not have any active staking activity

All stake deposit and rewards will be transfer to the new address

### Scenario 2: Transferring to an address which currently has 1 or more stake delegation or pending stake withdrawal

All stake deposits and rewards will be transferred to the new address. If there is any overlap with the existing stake delegation, rewards or pending withdrawal, the
transferred value will be added to the existing stake on the new address.

### Scenario 3: Cyclic transfer of stake

In this scenario, there are two addresses, address A and address B. Both addresses are transferring stake to each other.

```
A --> B
B --> A
```
**Case 1:** B accepts A's request first:
```
A --> (B) --> A
```
B accepts A's request; B inherits all of A's stake
Next, A accepts B request; A is final inheritor

**Case 2:** A accepts B's request first:
```
B --> (A) --> B
```
A accepts B's request; A inherits all of B's stake
Next, B accepts A's request; B is the final inheritor

However, there is no identified use case for cyclic transfer of a stake. Additionally, it may impact overall user experience. As such, such cyclic transfer will be disabled i.e., if the recipient is already a requestor in the `deleg_swap_request` map, transfer of stake request will not be possible till acceptance or cancellation of request. However, non-cyclic transfer will still be possible even if there is a pending stake transfer. 

### Mechanism 

The transfer will adopt a two-step process. First the transferer will need to initiate a request to transfer to another address. The recipient will then need to 
confirm the transfer. Upon confirmation, the transfer will be executed. 

Both parties must have zero buffered deposit and zero unwithdrawn rewards at the time of requesting and confirming the swap. 

4 new transitions in both `proxy` and `ssnlist` will be added to support this new feature.

| Transition | Comments |
| ---------- | -------- | 
| `RequestDelegatorSwap` | To initiate the request to transfer all existing stake deposit, rewards and pending withdrawals |
| `ConfirmDelegatorSwap` | To execute the transfer |
| `RevokeDelegatorSwap` | To cancel the transfer by the requestor |
| `RejectDelegatorSwap` | To cancel the transfer request from a specific requestor |

### Caveat

This mechanism does not support partial transfer.

### Other considerations 

Other than the intended purpose of transfer from address to address, the feature may open up a secondary market for stake transfer. For instance, an exchange
may choose to accept stake deposit transfer to its exchange wallet where the exchange can offer "unbonding" service by making use of their 
existing assets in their balance sheet. 

In such a case, the secondary market undertaking this transfer will need to take additional risk as the bonding period may be adjusted from time to time by the 
Zilliqa community via a governance vote. Also, the stake deposit holder will need to take on counterparty risk by transferring his stake to a third party. 

## Proper deletion of empty map entries

Additional checks are implemented to check for empty maps after a map deletion operation. This is done by calling various clean up procedures. This will incur 
additional gas costs to the delegator. Based on our experiments, the expected gas consumption is expected to increase by around 5%. 

We have also implemented the following transitions. They  will be used by the contract admin to clean up any empty map post state migration. 
- `CleanBuffDeposit`
- `CleanDirectDeposit`
- `CleanDelegStakePerCycle`
- `CleanDepositAmt`
- `CleanPendingWithdrawal`
- `CleanLastWithdrawCycle`
- `CleanLastBuffDepositCycle`

## Removal of custom ADT at `AssignStakeRewards` transition

To resolve the `disambiguation bug`, the custom ADT `SsnRewardShare` in `AssignStakeReward` transition parameters will be modified to `List (Pair ByStr20 Uint128))` 
in both `proxy` and `ssnlist` contracts.

## Migration of contract state

A new set of smart contracts will be deployed and populated with the states from the Phase 1 staking contracts. Both remote state read and populate transitions will be used to populate the Phase 1.1 staking contracts. 

### 1. Migration using remote state read
For maps with no user defined ADTs, remote state read will be used to read the states from the Phase 1 contract and populate it into the Phase 1.1 contract. The following maps will
be populated using this mechanism. 
- `comm_for_ssn`
- `deposit_amt_deleg`
- `ssn_deleg_amt`
- `buff_deposit_deleg`
- `direct_deposit_deleg`
- `last_withdraw_cycle_deleg`
- `last_buf_deposit_cycle_deleg`
- `deleg_stake_per_cycle`
- `withdrawal_pending`

### 2. Migration using populate transition  
For the remaining of the maps, the `populate*` transitions will be use to manually populate each map. 

### 3. Cleaning up of map with empty entries  
As Phase 1 contract states contain nested maps with empty entries, `clean*` transitions will be used after the population to clean up any empty maps. 

### Duration needed
The whole migration and verification of migration is expected to take up to 7 days to complete. Upon completion of the verification by the team, we will unpause the staking contract
and staking activities can resume. 
# Changes to staking parameters

In the upcoming Zilliqa `v8.0.0`, the block time is expected to be reduced as a result of various improvements within the core protocol. The block reward in `v8.0.0` will be adjusted accordingly to preserve the previous inflation rate. As such, the following parameters in the staking contract will be adjusted as follows:

| Parameter | Phase 1 | Phase 1.1 |
|-------------- | ------------- | --------- |
| 1 Cycle duration | ~27 hours | ~24 hours | 
| Block per cycle | 1800 | 2500 | 
| Reward per cycle | 1,980,000 $ZIL | 1,760,000 $ZIL |
| Unbonding period | 24,000 final blocks | 35,000 final blocks (~2 weeks) |

Please note that the above parameters depend heavily on Zilliqa `v8.0.0`, which will require more observation of Zilliqa mainnet. The above parameters change are considered
interim. If the parameters deviate from the estimate, a follow-up ZIP can be proposed to adjust the parameters.
# Contract Specification and Implementation

More details on the contracts can be found in the [staking contract repository](https://github.com/Zilliqa/staking-contract/tree/main/contracts).

For direct access:

* The specification for the different contracts needed for Phase 1.1 can be found in the [Readme](https://github.com/Zilliqa/staking-contract/blob/main/README.md)

* Removal of custom ADT for AssignStakeRewards [[1](https://github.com/Zilliqa/staking-contract/pull/218)] [[2](https://github.com/Zilliqa/staking-contract/pull/225)]

* [Swapping of address for delegator](https://github.com/Zilliqa/staking-contract/pull/219)

* [Proper deletion of empty maps](https://github.com/Zilliqa/staking-contract/pull/222)

* [State migration](https://github.com/Zilliqa/staking-contract/pull/224)

# Limitations and Future Work

ZIP-19 aims to make minor functional improvements and fixes to the non-custodial staking contract. 
Other major contracts enhancements will be left as a future work for future phases of staking. 

# Backward Compatibility

The non-custodial staking will be implemented as a new contract and the older
contract will be deprecated. Contract data will be migrated over to the new contract, 
through a migration exercise that is likely to last at least a week. The old contract will
be frozen permanently. 

## Copyright Waiver

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
