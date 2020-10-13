| ZIP | Title | Status | Type  | Author| Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) | 
| --- | ---------------------------- | ------ | ----- |----------------- | -------------------- |-------------------- | 
| 11   | Seed Node Staking Mechanism: Phase I | Implemented  | Standards Track |  Lulu Ren <lulu@zilliqa.com>, <br> Jun Hao Tan <junhao@zilliqa.com>, <br> Te Ye Yeo <teye@zilliqa.com>, <br> Han Wen Chua <hanwen@zilliqa.com>, <br> Mervin Ho <mervin@zilliqa.com>,  <br> Antonio Nunez <antonio@zilliqa.com>,  and <br> Amrit Kummer <amrit@zilliqa.com> | 2020-08-17|2020-10-13|

# Table of Content

- [Abstract](#abstract)
- [Background](#background)
- [Design Consideration for Phase I](#design-considerations-for-phase-i)
  * [Non-custodial Staking](#non-custodial-staking)
  * [Uncapped Staking](#uncapped-staking)
  * [Governance Tokens aka gZIL](#governance-tokens-aka-gzil)
    + [Issuance Mechanics](#issuance-mechanics)
    + [Usage](#usage)
- [Non-custodial Seed Node Staking Overview](#non-custodial-seed-node-staking-overview)
  * [Staking Parameters](#staking-parameters)
  * [Verifier](#verifier)
  * [Seed Node Operator](#seed-node-operator)
  * [Delegator](#delegator)
- [Contract Specification and Implementation](#contract-specification-and-implementation)
- [Limitations and Future Work](#limitations-and-future-work)
- [Backward Compatibility](#backward-compatibility)


# Abstract

ZIP-11 presents the Phase I extension of the seed node staking proposal as
presented in [ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md).
This new proposal introduces a _non-custodial_ mechanism to stake tokens with a
seed node operator via a Scilla contract. Non-custodial here meaning that any
tokens that need to be staked can now be deposited directly in the contract and
therefore need not go through any intermediary entity acting as a custodian.
ZIP-11 also introduces some key changes in the staking parameters such as
lifting the cap on the total stake deposit (a consequence of the tokenomic
changes as proposed in
[ZIP-9](https://github.com/Zilliqa/ZIP/blob/zip-9/zips/zip-9.md)).



# Background

We strongly recommend readers to go through
[ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md) and
[ZIP-9](https://github.com/Zilliqa/ZIP/blob/zip-9/zips/zip-9.md) as they form
the basis for the current proposal.

To summarize, ZIP-3 presents the key idea of _seed node staking_ --- a staking
mechanism to open up the _seed nodes_ for developers and the broader community.
Seed nodes are special nodes that do not participate in the consensus but
instead archive historical transaction data. Seed nodes are important to
provide services like explorer.

The proposal put forth in ZIP-3 sets aside 5% of the mining rewards for seed
node operators who in return are expected to archive all historical transaction
data. The architecture assumes a set of verifiers (currently the cardinality of
the set being 1), that check liveness and data availability by periodically
querying block data for randomly chosen block heights and by comparing the
response with the one returned by a "trusted" oracle.

In order to become a seed node operator, one has to stake a minimum of 10 mil
ZIL tokens. However, an operator that cannot meet the minimum requirement on
its own may accept tokens delegated to it by other token holders. The reward
earned by the operator is shared among its delegators. The seed node operator
may take a commission to cover its operational expenses.

With ZIP-3, delegation of tokens required transferring of tokens to a
client-unique address provided by the operator which pooled all the tokens and
then deposited them in a Scilla contract. The operator upon receiving rewards
(which gets distributed daily) then distributes back the reward to the
delegators in proportion to their stake.  

ZIP-9 somewhat complements ZIP-3 by proposing several key changes in the
tokenomic design of ZIL tokens. The most pertinent change for the proposal
herein is the increase of allocation of block reward for seed node operators
from 5% to 40%. With this increase, more delegators can be accommodated in the
system as more rewards become available.

# Design Considerations for Phase I 

## Non-custodial Staking

ZIP-3 (aka Seed Node Staking Phase 0) had two key limitations. The first being
that the delegation was custodial, i.e., delegators had to transfer their
tokens to a seed node operator (such as an exchange) which pooled tokens
together and then deposited them in a contract. The operator also collected
yields on delegators' behalf. Clearly, the seed node operator had the custodial
liability and therefore had to be trusted to keep the funds safe.

ZIP-11 (aka Seed Node Staking Phase I) aims to address this issue by proposing
a contract design that allows delegators to directly deposit their stake in a
contract thereby eliminating the need to send their tokens to a custodian.  The
contract now takes over the accounting responsibility to keep track of tokens
delegated by each delegator with each seed node operator. It also computes
on-chain the reward that each delegator should get which is proportional to the
number of tokens delegated.

>**Intermezzo: Self-custodial or Non-Custodial?** Even though there does not
seem to be a well-defined distinction between self-custodial asset management and
non-custodial management and are often used interchangeably in the blockchain
space, we argue that there is a difference between the two. <br> <br> Notice
that in both ZIP-3 and ZIP-11, assets leave the wallet of the delegator. In
case of ZIP-3, the tokens move from the delegator's wallet address to an
address (unique to the delegator) assigned and controlled by the operator.
Once enough tokens have been pooled together to meet the minimum stake deposit,
they are transferred to the contract. The safety of the assets held in the
contract relies on the security of the contract. <br> <br> In case of ZIP-11,
delegators can directly deposit their tokens into the contract by providing the
address of the operator with which they wish to stake. ZIP-11 removes the need
of any intermediate addresses.  ZIP-11 therefore provides a non-custodial
mechanism to delegate tokens. The non-custodial property comes from the fact
that there is no single entity that holds the asset on behalf of delegators at
any point of time. Assets are either in the hands of the delegator or held in a
contract on-chain, the mechanics of which is transparent to the public. <br>
<br> Compare this with a mechanism where assets do not leave the delegator's
wallet, for instance they get locked at the protocol-layer and therefore cannot
be moved and hence are considered staked.  The delegator always holds the
custody of the asset. We refer to this as _self-custody_. <br><br> ZIP-11 does
not provide a self-custodial staking mechanism but a non-custodial way to
delegate tokens.

## Uncapped Staking

The second limitation with ZIP-3 was related to the total number of tokens that
could ever be delegated. Given that only 5% of the block reward was available
to reward the operators, the more tokens got staked, the thinner the
distribution of the rewards became, thereby, potentially resulting in a
situation where the reward and the commission becomes so low that it could not
cover the operational expense of running a seed node. In order to tackle this
issue, ZIP-3 proposed a cap of 61 million tokens to be staked on each operator
and a total of 610 million tokens across all operators. This was to ensure that
the operator would get an yield of ~10% per year.

However, due an excessive market demand, the seed node operators had to stretch
beyond the 61 million cap. Due to the custodial nature of the contract design,
it was possible for the operators to accept say 2x61 million tokens but deposit
only 61 million in the contract but dividing the reward earned from 61 million
tokens to delegators contributing  2x61 million tokens, thereby reducing the
yield from ~10% to ~5% per year.

In light of this, ZIP-9 was proposed to increase the block reward allocated for
seed node rewards from 5% to 40%. With this change, it will become possible to
lift the cap of 610 million tokens and yet be able to provide a healthy yield.
ZIP-11 incorporates the change proposed in ZIP-9 thereby solving the issue of
oversubscription.

## Governance Tokens aka gZIL

This ZIP also introduces _gZILs_, short for _governance ZILs_. gZILs will be
ZRC-2 compliant fungible tokens that will be earned alongside staking rewards.
The rationale behind issuing gZILs is to capture long-term token holders and
give them access to governance tokens that they can later use to make
ecosystem-wide decisions.

In order for any governance mechanism to succeed, the system must be able to
identify, engage and equip (with the right tools) long-term token holders that
contribute actively to network activity and ecosystem growth and have enough
skin-in-the-game to make impactful governance decisions. Achieving this with
ZILs, more than 2 years after their initial issuance is difficult if not
impossible. Whence, the _raison d'être_ of gZILs. 

Furthermore, by bootstrapping the issuance of governance tokens gZILs with the
staking program, it will become easier to organically capture long-term token
holders. We elucidate this further in the following section. 



### Issuance Mechanics

Number of gZILs issued to delegators will aim to capture the following key
characteristics of stake holders:

1. _Stake holder strength_ measured by the quantum of stake in the system.
2. _Stake holder commitment and long-term belief_ measured by the staking
   duration. 
3. _Stake holder contribution to network activity and growth_ measured by how
   frequently they transact on the network.
4. _Early vs. later-stage stake holders_ measured by how early the stake
   holders get involved in the program. 

Since the staking reward earned (in ZILs) is proportional to the stake deposit
and the staking duration, it is easier to capture the first two characteristics
by tying the number of gZILs earned with the reward earned in ZILs. In other
words, the longer a delegator stakes her tokens and the larger her stake is,
the more gZILs she will earn.

However, to capture the contribution to network activity, we propose that gZILs
be issued only when a delegator manually withdraws her reward by sending a
transaction to the network. A delegator would withdraw the staking reward for a
wide variety of reasons such as to restake her reward to cumulatively earn more
rewards, to use it in a dapp, to pay for gas or for trading in the secondary
market. Issuing gZILs alongside stake rewards will capture the
(staking-related) network activity contributed by the delegator.

The last characteristic to capture is to identify and incentivize early birds.
To this end, we propose that **gZILs be issued only for 1 year**, with the
objective to create scarcity and incentivize the early birds to get involved in
the staking program. Furthermore, **we limit the maximum number of gZILs to be
ever issued to be 722,700**. This is 1/1000 of the total ZIL reward available
for staking on an annual basis (Cf. table in the next
[section](#staking-parameters)). In other words, for every 1000 ZIL earned as
staking reward, 1 gZIL will be issued. **gZILs will not be rewarded to SSN operators 
unless they themselves delegate their stake.** Earning commission does not entitle 
operators to gZILs.

Do note that it is very possible that all the 722,700 gZIL may not get minted
as the number of gZILs that get minted depends on the frequency of stake reward
withdrawals.   In a more general sense, the actual issuance curve of gZIL is
hard to predict due to its dependence on the number of ZILs staked in the
contract and the frequency of reward withdrawal by the delegators. Since
rewards earned are not automatically staked in a cumulative manner, delegators
will be required to manually withdraw their rewards, and in doing so will
receive gZILs. 


### Usage 

gZIL will play an important role in the governance of the staking program, say,
to decide on the value of certain staking parameters. We also plan to put gZILs
to use for broader ecoysystem governance for example with a DAO-like structure
that will invest in community projects. A [Gitcoin](https://gitcoin.co/)-like
DAO will be setup with funds from Zilliqa Research to fund ecosystem projects
and initiatives.  The end goal is to move move all ecosystem funding currently
done by Zilliqa Research (as a part of ZILHive) to the DAO, making the
community responsible for making decisions on funding ecosystem projects. The
community holding gZILs will be able to vote on proposals alongside Zilliqa
Research on making decisions. More on this will be released as a separate ZIP.

Since the first non-staking related utility of gZIL will be in voting in a DAO,
gZIL must capture token holders that are long-term ecosystem participants with
a deep-rooted interest in making the Zilliqa ecosystem grow and succeed.
Issuing gZIL alongside staking rewards aims to capture those token holders.

> _Market Value of gZILs_: gZILs will have no pre-defined exchange rate pegged
to ZIL, i.e., gZILs cannot be redeemed for ZILs. However, since gZILs will be
needed for ecosystem governance, we believe that a secondary market for gZIL
may open up on the upcoming [ZilSwap DEX](https://zilswap.io/swap) that will
help with the price discovery of gZIL.



# Non-custodial Seed Node Staking Overview

## Staking Parameters

As proposed in
[ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md), seed node
staking will not dilute the maximum token supply which remains fixed to 21
billion tokens. However, with
[ZIP-9](https://github.com/Zilliqa/ZIP/blob/zip-9/zips/zip-9.md) in place, the
Zilliqa protocol will now allocate 40% of block rewards (that gets disbursed to
the miners every hour or so) to reward seed nodes. The table below presents a
further breakdown pre-ZIP-9 and post-ZIP-9. Note that block rewards get
distributed at the end of each DS epoch.

| Mainnet parameter                                           | Pre-ZIP-9 Value            | Post ZIP-9 Value             |
| -----------------------------------------------------------| ----------------------------| ---------------              |
| Total mining reward distributed per DS epoch  (in ZIL)      | 197,244                    | 275,000                      |
| Average duration of a DS epoch (in mins)                    | 83                         | 83                           |
| Number of DS epochs per day                                 | 17                         | 18                           |
| Number of DS epochs per year                                | 6,205                      | 6,570                        |
| Percentage of reward proposed for seed nodes                | 5%                         | 40%                          |
| Total reward available for seed nodes per DS epoch (in ZIL) | 197,244 x 0.05 = 9,862     | 275,000 x 0.4 = 110,000      |
| Total reward available for seed nodes per year (in ZIL)     | 6,205 x 9,862 = 61,193,710 | 6,570 x 110,000 = 722,700,000|

As shown in the table above, if 40% of block reward goes to the seed nodes,
then a total of ~722 million ZILs per year can be used to provide the necessary
incentives. With this total reward available, we propose the following economic
parameters for staking:

| Staking parameter                            | ZIP-3 Value           | ZIP-11 Value |
| -------------------------------------------  | --------------------- |-----------------          |
| Maximum overall staked amount (in ZIL)       | 610,000,000           | Uncapped                  |
| Maximum stake amount (in ZIL) per seed node  | 61,000,000            | Uncapped                  |       
| Minimum stake amount (in ZIL) per seed node  | 10,000,000            | 10,000,000                |
| Minimum stake amount (in ZIL) for delegators |        NA             | 10                        |
| Maximum number of seed nodes                 | 10                    | 30                        |
| Reward cycle                                 | 17 DS blocks (~1 day) | 18 DS block (~1 day)      |
| Maximum commission rate change per cycle     |        NA             | < 2%                      |
| Annual interest rate                         | 10.03%                | Variable                  |
| Stake unbonding period                       |  NA                   | 24,000 blocks (~14 days)  |
| Reward unbonding period                      |  NA                   | NA                        |

The rationale behind introducing a minimum stake amount for delegators is to
ensure that the staked reward is not less than the gas paid to withdraw it.

**Unbonding Period:** ZIP-11 also introduces the notion of _unbonding period_
for stake withdrawals. Once a delegator has made a request for her stake
withdrawal, she will have to wait for the unbonding period (initially proposed
to be roughly 2 weeks) to expire. Upon expiry of the unbonding period, the
delegator may claim back the tokens which get moved from the contract to the
delegator's address. Note that the unbonding period will be a mutable value and
hence it can be changed through community governance via gZILs. The delegator
does not earn rewards during the unbonding period.
 

There is no unbonding period for reward withdrawals though. Any reward earned
through staking can be withdrawn as soon as they have been distributed.   

## Verifier

The role of the verifier in ZIP-11 is the same as the one in ZIP-3. The
(trusted) verifier which sits off-chain periodically checks the health of
each SSN node for example by querying for random transaction data via the
public APIs. See [ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md#c-staking-proofs) 
for more details on the checks. For each SSN, it computes `verification_passed` 
which is the percentage of tests that the SSN has passed.

The reward earned (in ZIL) by a given SSN is then computed by taking into
account the total reward available for seed nodes per DS epoch (which is
110,000 Cf. table above), the total number of DS epoch per reward cycle
(roughly 17) and the verification success rate in percentage. This reward is
then distributed in proportion to the stake deposited, hence the factor
`(TotalStakeAtSSN / TotalStakeAcrossAllSSNs)`.

```ocaml
SSNRewardForCurrentCycle = floor((NumberOfDSEpochsInCurrentCycle x 110,000 x VerificationPassed)) x floor(TotalStakeAtSSN / TotalStakeAcrossAllSSNs)

```

The first part of the computation `floor(NumberOfDSEpochsInCurrentCycle x
110,000 x VerificationPassed)` is computed off-chain by the verifier, while the
factor `floor(TotalStakeAtSSN / TotalStakeAcrossAllSSNs)` is computed on-chain
within the smart contract by using the most updated data which is stored as a
part of the contract state.

The verifier computes the off-chain part as an integer value for every cycle
and calls the following transition in the contract to compute stake reward for
each reward cycle.

```ocaml
 transition AssignStakeReward(ssnreward_list: List SsnRewardShare, verifier_reward: Uint128)

```

The first parameter of the transition is a list of a data type named
`SsnRewardShare` which is essentially a pair of types `(ByStr20, Uint128)` that
represents `(SSNAddress, SSNRewardForCurrentCyle)`. The first element of the
pair is the address of the SSN, while the second element is the off-chain value
computed by the verifier for this SSN: `floor(NumberOfDSEpochsInCurrentCycle x
110,000 x VerificationPassed)`. The transition iterates over all the SSNs and
computes the factor `floor(TotalStakeAtSSN / TotalStakeAcrossAllSSNs)` and
assigns reward to each SSN. A small percentage of these rewards goes to the SSN
operators in the form of commission to cover operational expenses while the
remaining bulk is to reward the delegators.

In the case where, the total reward meant to be distributed to the seed nodes
cannot be assigned to seed nodes (owing to poor performance of any of the seed
nodes or more concretely, when `VerificationPassed` is not 100% for any of the
SSNs), then the left-over reward is given to the verifier. This is captured via
the parameter `verifier_reward` passed in the transition.

## Seed Node Operator

We intend to start with 10 seed node operators and revisit the number in the
future. Each seed node operator will have to be registered by the smart
contract admin. Once registered, it can invoke the following transitions in the
proxy contract:

1. `transition UpdateComm(new_rate: Uint128)` to update the commission. The
   contract puts some restrictions on the frequency of commission rate updates
to dissuade unscrupulous operators from advertising a low commission to attract
delegators and once the delegators have delegated their stake later increase
the commission rate to a high value. The contract restricts operators from
changing the commission more than once during the same reward cycle (i.e,
within a day). Additionally, to avoid drastic spike or drip in the commission
rate for SSN operators, the change in the commission per cycle is bounded by
`maxcommchangerate` (initially set to 1%).
2. `transition WithdrawComm(ssnaddr: ByStr20)` to withdraw the commission
   earned.
3. `transition UpdateReceivingAddr(new_addr: ByStr20)` to update the address to
   receive commission.

## Delegator

Delegators can stake their ZILs by directly calling the contract. They can call
the following transitions:

1. `transition DelegateStake(ssnaddr: ByStr20)` to delegate their funds to a
   specific SSN. As mentioned earlier, a delegator must stake a minimum of 1000
ZILs. This to ensure that the gas needed to withdraw the reward does not
outweigh the reward itself. If the SSN is active (i.e., 10 mil ZILs have been
already staked with this SSN and that it is already operational), then the
deposited stake will be buffered for a cycle and will be included in the SSN's
stake pool in the next cycle. If the SSN is inactive, then the stake amount
deposited can be directly included as a part of the SSN's stake pool.

2. `transition WithdrawStakeRewards(ssn_operator: ByStr20)`to withdraw their
   stake rewards from a specific SSN and mint gZIL tokens.  Re-delegation of
stake rewards is manual and a two-step process. First, the delegator has to
withdraw the rewards and then it will have to delegate the withdrawn reward as
stake. Wallet providers can make this user experience seamless.

3. `transition WithdrawStakeAmt(ssn: ByStr20, amt: Uint128)` to withdraw a
   specific amount from the stake. By calling this transition, the delegator
enters into the unbonding period. During this period, the delegator does not
earn any reward and her stake is held in the contract.  Upon expiry of the
unbonding period, the delegator can call the next transition
`CompleteWithdrawal` to move funds out of the contract.  

4. `transition CompleteWithdrawal()` to withdraw unbonded stake from the
   contract. 

5. `transition ReDelegateStake(ssnaddr : ByStr20, to_ssn : ByStr20, amount :
   Uint128)` to move `amount` stake from the current SSN (`ssnaddr`) to a
different SSN (`to_ssn`). 

# Contract Specification and Implementation

More details on the contracts can be found in the [staking contract
repository](https://github.com/Zilliqa/staking-contract/blob/dev/contracts/README.md).
For direct access:


* The specification for the different contracts needed for Phase I can be found [here](https://github.com/Zilliqa/staking-contract/blob/dev/contracts/README.md). 

* The implementation of the contracts can be found [here](https://github.com/Zilliqa/staking-contract/tree/dev/contracts). 

# Limitations and Future Work

While, ZIP-11 makes improvements over ZIP-3 in terms of providing a
non-custodial way for token holders to delegate their stake with an SSN. It
does not address the trust component that a single verifier brings. We list
below two key improvements for the next phase of seed node staking.

* **Detecting Cheating/Malicious Operators:** In **Phase 1** as in **Phase 0**,
  the Verifier implements rather simple checks to monitor the health of a seed
node, such as checking if the seed node holds data for randomly chosen blocks
and is alive when a fetch request (for a block data) is made. One possible
improvement could be to implement a [Proof of Retrievability
protocol](http://www.arijuels.com/wp-content/uploads/2013/09/BJO09b.pdf) - a
protocol that runs between a client and a data storage provider that guarantees
that the data storage provider indeed holds a certain data that the client has
outsourced to the storage provider.

* **Decentralizing Verifiers:** Another area for improvement in the next phases
  would be to have a decentralized layer of Verifiers, where any node can
potentially become a Verifier node and monitor seed nodes and report any Proof
of Poor Service (or PoPS) and get rewarded for it. Such designs have been
extensively explored in the past for example in
[TrueBit](https://people.cs.uchicago.edu/~teutsch/papers/truebit.pdf).

# Backward Compatibility

The non-custodial staking will be implemented as a new contract and the older
contract will be deprecated. This ZIP also introduces some backward
incompatible changes inherited from ZIP-9 particularly the 40% block reward
allocation.

## Copyright Waiver

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
