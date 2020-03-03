| ZIP | Title                        | Status | Type  | Author                                                                                                                       | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) |
| --- | ---------------------------- | ------ | ----- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------- | -------------------- |
| 3   | Seed Node Staking Mechanism | Draft  | Standards Track | Clark Yang <clark@zilliqa.com> <br> Sandip Bhoir <sandip@zilliqa.com> <br> Antonio Nunez <antonio@zilliqa.com> | 2020-01-30           | 2020-01-30           |

## Abstract

ZIP-3 defines a staking mechanism at the core protocol to promote and regulate the participation of _seed node_ hosts in the Zilliqa network.

## Background & Motivation

The Zilliqa network architecture consists of several types of nodes with different functionalities and responsibilities. One of such nodes is called the _seed nodes_. These nodes do not validate transactions and as a result do not run the consensus protocol. Seed nodes can therefore be considered as an ancillary set of nodes that play a supporting role in the overall Zilliqa network architecture.

The main role of seed nodes is to serve as direct access points (for end users and clients) to the core Zilliqa network that validates transactions. As "archival nodes", they further maintain the entire transaction history and the global state of the blockchain, which is needed to provide services such as block explorers. They also consolidate transaction requests and forward these to the _lookup nodes_ (another type of node) for distribution to the shards in the network. Seed nodes therefore present an ancillary yet critical component in the network architecture. 

Seed nodes are currently being run by several major exchanges, application providers such as [Xfers](https://www.xfers.com/sg/), the Zilliqa Team itself and the block explorer provider [ViewBlock](https://viewblock.io/zilliqa). Most of these seed nodes are however "closed", i.e., they only serve the needs of the seed operator and not the broader community.

In order to "open up" some of these nodes and decentralize the overall seed node architecture, a proper incentive mechanism must be put in place. With the right monetary incentive, seed nodes can potentially be hosted by any entity including, say, wallet providers, as well as the entire community at large. In the interest of decentralization, a means to facilitate the addition and management of seed nodes must also be implemented within the protocol. In order to maintain the overall health of the network, it is also essential to establish a minimum performance threshold for these seed nodes.

Staking provides one approach to both promoting widespread participation and ensuring an acceptable level of performance from participants. The idea of staking is to pre-qualify seed hosts by requiring them to stake a certain amount of ZILs for the duration of the service provided. Within this duration, the host presents regular proofs of its ability to provide the service. In return, the hosts are rewarded a proportional amount of ZILs in a predetermined manner.

## Seed Node Staking Phase 0: Design Considerations

This ZIP presents the first phase (dubbed **Phase 0**) for implementing staking for seed nodes. In this first phase of implementation, the main idea is to implement staking via a smart contract without slashing, i.e., node operators who wish to host a seed node will have to deposit a certain minimum number of ZILs in a smart contract by transferring the tokens to the contract. 

In return, if the nodes provide the expected service, they will be granted a part of the block rewards. In the scenario where a seed node operator is unable to provide the basic minimum service, the deposited stake will not be slashed. Instead, nodes providing poor service will forfeit the reward. The staking architecture has a _Verifier_ node that periodically checks whether a seed node operator is indeed providing the service for which it is rewarded.

A seed node operator that does not have the minimum number of ZILs to stake may open up this service to other token holders who may not have the right expertise to run a seed node themselves. The way in which the custody of tokens is handled is left to the seed node operator.

Due to these design considerations, **Phase 0** is not extremely intrusive to the core protocol. However, it does present some areas for improvement which will be the goal of the next phases. For instance, **Phase 0** requires seed node operators to deposit the stake to a smart contract. One possible improvement would be to not require an operator to transfer funds but instead lock those funds at the core protocol level, i.e., funds do not move but only stay locked. This will require handling the entire staking architecture at the protocol level, which, due to its intrusive nature, can be handled in the next phase based on the success of **Phase 0**.

In **Phase 0**, the Verifier implements rather simple checks to monitor the health of a seed node, such as checking if the seed node holds data for randomly chosen blocks and is alive when a fetch request (for block data) is made. One improvement that can be implemented in the next phase is to implement a [Proof of Retrievability protocol](http://www.arijuels.com/wp-content/uploads/2013/09/BJO09b.pdf) - a protocol that runs between a client and a data storage provider that guarantees that the data storage provider indeed holds a certain data that the client has outsourced to the storage provider.

Another area of improvement in the next phases would be to have a decentralized layer of Verifiers, where any node can potentially become a Verifier node and monitor seed nodes and report any proof of poor service and get rewarded for it. Such designs have been extensively explored in the past for example in [TrueBit](https://people.cs.uchicago.edu/~teutsch/papers/truebit.pdf).

An alert reader may argue that requiring seed nodes to stake may not be necessary. Instead, they could simply be rewarded for the service they provide. **Phase 0**, however, includes staking so as to prepare for the later phases when slashing will be implemented. Slashed ZILs can also be given to the Verifiers who monitor and fish for inactive seed nodes.

## Specification

### A. Terminology and Roles

| Name                         | Description                                                         |
| ---------------------------- | ------------------------------------------------------------------- |
| Staked Seed Node (SSN)       | The seed node that joins the network through this staking mechanism |
| Verifier                     | A dedicated machine for monitoring SSNs                             |

### B. Setup

The diagram below shows the overall setup. In particular, the colored areas suggest the components that must be created for this staking mechanism.

![image01](../assets/zip-3/image01.png)

The setup includes the following:

#### Within Zilliqa Research
1. A Verifier node monitors the performance of SSNs and dispenses rewards accordingly through smart contract transactions.
2. A series of multipliers forward mined blockchain data from the mainnet to a whitelist of IP addresses.
3. A set of lookups provide the reference data to the Verifier, including publicly accessible data (through [api.zilliqa.com](https://api.zilliqa.com/)) and raw data with access limited to the Verifier.

#### Within Hosting Entity
1. A SSN node hosts a `zilliqa` instance that receives blockchain data from the multipliers.
2. The same node also hosts a `ssn` instance that retrieves raw data from the `zilliqa` instance for the purpose of presenting proofs to the Verifier

### C. Staking Proofs

There are three things that a SSN must satisfy in order to receive staking rewards:
1. It must be recognized as an active SSN in the staking smart contract.
2. It must pass the checks for raw data storage requested by the Verifier.
3. It must pass the checks for servicing API requests by the Verifier.

#### SSN Activation

SSN registration is performed during an initial handshake process between the Verifier and the SSN and concludes with the addition of the SSN into the staking smart contract.

SSN activation, which can be done anytime after, requires the SSN host to deposit into the contract an amount satisfying both the minimum and maximum stake criteria (invalid amounts are rejected from being deposited).

#### Storage and API Checks

The Verifier polls the SSN for raw blockchain data and API-retrievable data to effectively confirm that the SSN both stores the blockchain data and services API requests from end users.

Raw blockchain data is accessible from the lookups on the Verifier's end and from the `zilliqa` process on the SSN host's end.

The diagram below shows the interaction between the different components for the storage and API checks.

![image02](../assets/zip-3/image02.png)

### D. Staking Smart Contract

As part of this staking mechanism, a smart contract named `SSNList` is deployed to facilitate the rewarding scheme and maintain the list of SSNs.

#### Roles

| Name            | Description                                       |
| --------------- | ------------------------------------------------- |
| `verifier`      | The Verifier node                                 |
| `contractadmin` | A user with administrator access to this contract |
| `ssn`           | A registered SSN node                             |

#### Immutable Variables

| Name        | Type      | Description             |
| ----------- | --------- | ----------------------- |
| `verifier`  | `ByStr20` | Address of the Verifier |

#### Mutable Fields

| Name        | Type                                | Description                                        |
| ----------- | ----------------------------------- | -------------------------------------------------- |
| `ssnlist`   | `Map ByStr20 Ssn = Emp ByStr20 Ssn` | Mapping between SSN address and the following information: <br> - `alive_status` (`Uint32`) <br> - `stake_amount` (`Uint128`) <br> - `rewards` (`Uint128`) <br> - `blocknumber_when_alive_status_updated` (`BNum`) <br> - `ip_addr` (`String`)|
| `minstake`  | `Uint128`                           | Minimum `stake_amount` required to activate a SSN |
| `maxstake`  | `Uint128`                           | Maximum `stake_amount` allowed to activate a SSN |
| `contractadmin` | `Option ByStr20  = None {ByStr20}` | Address of user with administrator access to this contract |

#### Transitions

##### 1. update_minstake

```ocaml
(* @dev: Set the minstake of contract. Used by verifier only. *)
(* @param min_stake: New minstake value *)
transition update_minstake (min_stake : Uint128)
```

##### 2. update_maxstake

```ocaml
(* @dev: Set the maxstake of contract. Used by verifier only. *)
(* @param max_stake: New maxstake value *)
transition update_maxstake (max_stake : Uint128)
```

##### 3. update_admin

```ocaml
(* @dev: Set the admin of contract. Used by verifier only. *)
(* @param min_stake: New admin value *)
transition update_admin (admin : ByStr20)
```

##### 4. deposit_funds

```ocaml
(* @dev: Move token amount from _sender to recipient i.e. contract address. *)
transition deposit_funds ()
```

##### 5. stake_deposit

```ocaml
(* @dev: Moves an amount tokens from _sender to the recipient. Used by token_owner. i.e. ssn *)
(* @dev: Stake amount of exisitng ssn in ssnlist will be updated with new amount only if existing stake amount is 0. 
         Balance of contract account will increase. Balance of _sender will decrease.      *)
transition stake_deposit ()
```

##### 6. add_ssn

```ocaml
(* @dev: Adds new ssn to ssnlist. Used by verifier only. *)
(* @param ssnaddr: Address of the ssn to be added *)
(* @param ssnip: string representing "ip:por" of the ssn to be added *)
(* @param blocknumber: Block number when the verifier invoked this transition *)
transition add_ssn (ssnaddr : ByStr20, ssnip : String, blocknumber : BNum)
```

##### 7. update_ssn_liveness

```ocaml
(* @dev: update the liveness status of ssn. Used by verifier only. *)
(* @param ssnaddr: Address of the ssn whose liveness is to be updated *)
(* @param liveness: New liveness of the ssn to be updated *)
(* @param blocknumber: Block number when the verifier invoked this transition *)
transition update_ssn_liveness (ssnaddr : ByStr20, liveness : Uint32, blocknumber : BNum)
```

##### 8. assign_stake_reward

```ocaml
(* @dev: Assign stake reward to specific ssn from ssnlist. Used by verifier only. *)
(* @param ssnaddr: Address of the ssn to be awarded the reward *)
(* @param reward_percent: reward share awarded to ssn *)
transition assign_stake_reward (ssnaddr : ByStr20, reward_percent : Uint128)
```

##### 9. withdraw_stake_rewards

```ocaml
(* @dev: Withdraw stake reward. Used by ssn only. *)
transition withdraw_stake_rewards ()
```

##### 10. withdraw_stake_amount

```ocaml
(* @dev: Move token amount from contract account to _sender. Used by ssn only. *)
(* @param amount: token amount to be withdrawed *)
transition withdraw_stake_amount (amount : Uint128 )
```

##### 11. remove_ssn

```ocaml
(* @dev: Remove a specific ssn from ssnlist. Used by verifier only. *)
(* @param ssnaddr: Address of the ssn to be removed *)
transition remove_ssn (ssnaddr : ByStr20)
```

##### 12. drain_contract_balance

```ocaml
(* @dev: Set the admin of contract. Used by verifier only. *)
(* @param min_stake: New admin value *)
transition drain_contract_balance ()
```

### E. Staking Procedure

#### Step 1 - Zilliqa Research sets up the staking components

A dedicated machine running the `verifier` process is launched. A key pair is assigned to the Verifier.

Zilliqa Research lookup nodes are configured to recognize the Verifier key, for use during JSONRPC requests via port 4401.

The `SSNList` smart contract is deployed and initialized with the address of the Verifier. The `update_minstake`, `update_maxstake`, and `update_admin` transitions are called to initialize the staking settings. The `deposit_funds` transition is also called to fund the contract for the rewards distribution.

#### Step 2 - Host registers with Zilliqa Research

The entity intending to host a SSN registers with the Zilliqa Research team. Zilliqa Research assesses the suitability of the operator to host a SSN. After the assessment, the registration process includes whitelisting the IP address so that the multipliers and lookup nodes can communicate with the SSN.

#### Step 3 - Host launches SSN

The `zilliqa` process is launched in the host machine in the manner typical of a seed node. As a typical seed node, the SSN should be able to service API requests at this point through the host's own API domain (akin to [api.zilliqa.com](https://api.zilliqa.com)).

Additionally, a separate `ssn` process is launched in the same machine. A key pair is assigned to the `ssn`. The `zilliqa` process launched earlier should have been configured to recognize this key, for use during JSONRPC requests via port 4401.

Upon startup, the `ssn` process initiates contact with the Verifier node.

#### Step 4 - Verifier adds SSN to smart contract

The Verifier and SSN perform the initial checks on both the blockchain data stored in the SSN as well as the SSN's ability to respond to API requests. Upon successful completion, the Verifier calls the `add_ssn` transition in the smart contract, which creates an entry for the SSN in the `ssnlist` table in the contract. The `ip_addr` in the entry is initialized to the IP address of the SSN. The `blocknumber_when_alive_status_updated` is initialized to the Tx epoch number when the entry was created.

#### Step 5 - Host deposits funds for staking

A SSN in the `SSNList` contract with `stake_amount` falling below `minstake` is not considered active at this point, and is ignored by the Verifier. The host must activate its SSN by depositing funds into the contract through a single call to the `stake_deposit` transition.

> Note: Stake fund depositing can only be performed once. In order to change the staked amount, the host will need to de-register and register back the SSN.

#### Step 6 - Verifier performs regular checks

The Verifier accesses the list of SSNs in the smart contract. For each active SSN, it performs the storage and API servicing checks at periodic intervals. The result of each check is recorded by the Verifier by calling the `update_ssn_liveness` transition in the smart contract. If the check is successful, the transition updates (i.e., increments) the `alive_status` information for the SSN in the `ssnlist` table.

#### Step 7 - Verifier distributes rewards

For each SSN, the Verifier calls the `assign_stake_reward` transition in the smart contract at periodic intervals to trigger the rewards distribution. The reward amount is added to the `reward` for the SSN in the `ssnlist` table. The `alive_status` value and number of verification runs are also reset afterwards, readying the SSN for the next verification and rewarding cycle. See the [Rewarding Algorithm](#f-rewarding-algorithm) section for the details.

#### Step 8 - Host withdraws rewards

A host may at any time withdraw any available funds from its SSN by calling the `withdraw_stake_rewards` transition in the smart contract. This transition transfers the reward amount from the smart contract balance to the address of the SSN.

#### Step 9 - Host de-registers as a SSN

If an entity decides to conclude its participation as a SSN host, it does so by performing the following in sequence:
1. calling the `withdraw_stake_rewards` transition in the smart contract to withdraw the entire `reward` amount
2. calling the `withdraw_stake_amount` transition in the smart contract to withdraw the entire `stake_amount` amount

The second step above automatically removes the SSN from the `ssnlist`.

> Note: Although the smart contract also contains a `remove_ssn` transition, this is not intended for normal situations and can only be invoked by the Verifier when necessary.

Finally, the entity requests the Zilliqa Research team to remove the IP address of the SSN from the network's whitelist.

### E. Verification and Rewarding Frequencies

SSN verification involves the checks listed in the [Staking Proofs](#c-staking-proofs) section.

Verification is initially performed during the handshake period between the Verifier and the SSN, which ends with the addition of the SSN (still inactive at this point) into the smart contract.

After SSN activation, verification is done a maximum of `NUM_OF_RUNS_EACH_REWARD_CYCLE` times within the span of `NUM_OF_DSBLOCK_EACH_REWARD_CYCLE` DS epochs, with the verification runs done at randomized intervals. Each verification result is recorded within the smart contract (i.e., through the `alive_status` field). The Verifier tracks the number of verification runs performed during the current reward cycle, and this count is applied across all SSNs.

After `NUM_OF_DSBLOCK_EACH_REWARD_CYCLE` DS epochs, the Verifier calls the smart contract to trigger the rewards distribution.

### F. Rewarding Algorithm

The Verifier is configured to dispense `EFFECTIVE_INTEREST_RATE` as the effective interest rate per rewarding cycle. This has to be set based on the annual interest rate divided by the expected number of reward cycles within a year. That is:

```
EFFECTIVE_INTEREST_RATE = annual interest rate / reward cycles per year
```

SSNs are rewarded based on the percentage of successful verification runs performed within the `NUM_OF_DSBLOCK_EACH_REWARD_CYCLE` cycle. This means that a SSN earns the maximum rewards if its `alive_status` in the smart contract is equal to the total number of verification runs performed across all SSNs within the cycle. A SSN can potentially earn less than that if it either fails one or more verification runs or if it joins in the middle of a rewarding cycle and is unable to undergo the full number of verification runs.

When the Verifier calls the `assign_stake_reward` transition to perform the rewarding, the `reward_percent` parameter is thus set as:

```
reward_percent = EFFECTIVE_INTEREST_RATE x (alive_status / number of verification runs performed)
```

## Rationale

**Phase 0** staking mechanism design leverages on the existing infrastructure and therefore requires minimal changes as opposed to alternatives. Improvements to the design, such as time-sensitive and cryptographically stronger storage proofs, can be explored in future versions as explained in the [Design Considerations](#seed-node-staking-phase-0-design-considerations) section.

## Backward Compatibility

The staking mechanism is intended to work alongside the existing core protocol and should be fully backward compatible with all its components, i.e., no change in behavior should be observable on the mainnet operation with this mechanism in place.

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

