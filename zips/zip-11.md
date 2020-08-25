| ZIP | Title | Status | Type  | Author| Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) | 
| --- | ---------------------------- | ------ | ----- |----------------- | -------------------- |-------------------- | 
| 11   | Seed Node Staking Mechanism: Phase I | Draft  | Standards Track | Han Wen Chua
<hanwen@zilliqa.com>, <br> Mervin Ho <mervin@zilliqa.com>, <br> Lulu Ren
<lulu@zilliqa.com>, <br> Jun Hao Tan <junhao@zilliqa.com>, <br> Antonio Nunez
<antonio@zilliqa.com>,  and <br> Amrit Kummer <amrit@zilliqa.com> | 2020-08-17|
2020-08-17|


# Abstract

ZIP-11 is the Phase I extension of the seed node staking proposal as presented
in [ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md). This new
proposal introduces a non-custodial mechanism to stake tokens with a seed node
operator via a Scilla contract. Non-custodial here meaning that any tokens that
need to be staked can now be deposited directly in the contract and therefore
need not go through any intermediary entity acting as a custodian. ZIP-11 also
introduces some key changes in the staking parameters such as lifting the cap
on the total stake deposit (a consequence of the tokenomic changes as proposed
in [ZIP-9](https://github.com/Zilliqa/ZIP/blob/zip-9/zips/zip-9.md)).

# Background

Before reading any further, we strongly recommend readers to go through
[ZIP-3](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-3.md) and
[ZIP-9](https://github.com/Zilliqa/ZIP/blob/zip-9/zips/zip-9.md). 

To summarize, ZIP-3 presents the key idea of _seed node staking_ --- a staking
mechanism to open up the _seed nodes_ for developers and the broader community.
Seed nodes are special nodes that do not participate in the consensus but
instead archive historical transaction data. Seed nodes are important to
provide services like explorer. 

The proposal put forth in ZIP-3 sets aside 5% of the mining rewards to reward
seed node operators who in return are expected to archive all historical
transaction data. The architecture assumes a set of verifiers (currently the
cardinality of the set being 1), that periodically check data availability by
periodically querying for block data for randomly chosen blockheights and by
comparing the response with the one returned by a "trusted" oracle. 

In order to become a seed node operator, one has to stake a minimum of 10 mil
ZIL tokens. However, operators who cannot meet the minimum requirement on their
own have the possibility to allow other token holders to delegate their tokens
to the operator and in return earn rewards. The seed node operator may take a
commission to cover its operational expenses. 

Delegation of tokens required token holders to transfer their tokens to a
client-unique address provided by the seed node operator which pooled all the
tokens and the deposited them in a Scilla contract. The operator upon receiving
rewards (which gets distributed daily) then distributes back the reward to the
delegators in proportion to their stake.  

ZIP-9 proposes several key changes in the tokenomic design of ZIL tokens. The
most pertinent change for the proposal herein is the increase of allocation of
block reward for seed node operators from 5% to 40%.

# Design Considerations

## Non-custodial Staking

ZIP-3 (aka Seed Node Staking Phase 0) had two key limitations. The first being
that the delegation was custodial, i.e., token holders had to transfer their
holdings to a seed node operator (such as an exchange) which then deposited the
pooled tokens in a contract. The operator also collected yields on token
holders' behalf, thereby creating custody risks.

ZIP-11 (aka Seed Node Staking Phase I) aims to address this by proposing a
contract design that allows token holders to directly deposit their stake in a
contract thereby eliminating the need to send their tokens to a custodian. The
contract now takes over the accounting responsibility to keep a track of tokens
delegated by each user with each seed node operator. It also computes on-chain
the reward that each delegator should get which is proportional to the number
of tokens delegated by a holder. 

>Note: **Self-custodial or Non-Custodial?** <br>Even though there does not seem
>to be well-defined distinction between
self-custodial asset management and non-custodial management and are often used
interchangeably in the blockchain space, we argue that there is a difference
between the two. <br> Notice that in both ZIP-3 and ZIP-11, assets leave the
wallet of the token holder. In case of ZIP-3, the tokens move from the holder's
wallet address to an address (unique to the holder) assigned by the operator.
Once enough tokens have been pooled to meet the minimum stake deposit, the
tokens get pooled from the different addresses (provided by the operator to the
holders) into one single wallet address. All the pooled tokens then get
transferred from this pool address to the contract. The safety of the assets
held in the contract relies on the security of the contract. <br> In case of
ZIP-11, tokens holders can directly deposit their tokens into the contract by
providing the information on the address of the operator with which they wish
to stake. ZIP-11 removes the need of the intermediate addresses.  ZIP-11
therefore provides a non-custodial mechanism to delegate tokens. The
non-custodial property comes from the fact that there is no single entity that
holds the asset on behalf of holders at any point of time. Assets are either in
the hands of the holder or held in a contract on-chain. This is in line with
all the DeFi applications. <br> Compare this with a mechanism where assets do
not leave the holder's wallet. The holder always holds the custody of the
asset. We refer to this as _self-custody_. <br> ZIP-11 does not provide a
self-custodial staking mechanism but a non-custodial way to delegate tokens. 

## Uncapped Staking

The second limitation with ZIP-3 was related to the total number of tokens that
could ever be delegated. Given that only 5% of the block reward were available
to reward the operators, the more tokens get staked, the thinner the
distribution of the rewards becomes potentially resulting in a situation where
the reward and the commission becomes so low that it could not cover the
operational expense of running a seed node. In order to tackle this issue,
ZIP-3 proposed a cap of 61 million tokens to be staked on each operator and a
total of 610 million tokens across all operators. This was to ensure that the
operator would be get an yield of ~10% per year. 

However, due an excessive demand in the market, the seed node operators had to
stretch beyond the 61 million cap. Due to the custodial nature of the contract
design, it was possible for the operators to accept 2x61 million tokens but
deposit only 61 million in the contract but dividing the reward earned from 61
million tokens to delegator contributing  2x61 million tokens, thereby reducing
the yield from ~10% to ~5% per year.

In light of this, ZIP-9 was proposed to increase the block reward allocated for
seed node rewards from 5% to 40%. With this change, it will become possible to
lift the cap of 610 million tokens and yet be able to provide a healthy yield.
ZIP-11 incorporates the change proposed in ZIP-9 thereby solving the issue of
oversubscription.

## Governance Tokens aka gZIL

This ZIP also introduces gZILs (short for _governance ZILs_). gZILs will be
ZRC-2 compliant fungible tokens that will be earned alongside staking rewards.
More concretely, whenever, a delegator decides to withdraw her stake rewards,
an equivalent number of gZILs will be minted and issued to the delegator.
Since, the reward earned is proportional to the stake deposit and the staking
duration, the number of gZILs earned by a delegator will capture the long-term
belief of token holders. In other words, the longer a delegator stakes her
tokens and the larger her stake is, the more gZILs she will earn.

gZILs will later be used to govern a DAO-like structure that will invest in
community projects. A Gitcoin-like DAO will be setup with funds from Zilliqa
Research to fund ecosystem projects and initiatives. The end goal is to move
move all ecosystem funding currently done by Zilliqa Research (as a part of
ZILHive) to the DAO making the community responsible for making decisions on
funding ecosystem projects. The community holding gZILs will vote on proposals
instead of Zilliqa Research making decisions. More on this will be released as
a separate ZIP.

Since the purpose of gZIL will mainly be in voting in a DAO, gZIL must capture
token holders that are long-term ecosystem participants with a deep-rooted
interest in making the Zilliqa ecosystem grow and succeed. Issuing gZIL
alongside staking rewards aims to capture those token holders.

The issuance curve of gZIL is hard to predict given that it depends on the
number of ZILs staked in the contract and the frequency of reward withdrawal by
token holders. Since, rewards earned are not automatically staked in a
cumulative manner, token holders will have to manually withdraw their rewards
and in doing so they will receive gZILs.

gZIL will have no pre-defined exchange rate pegged with ZIL, i.e., gZILs cannot
be redeemed for ZILs. However, since gZILs will be needed to vote in the DAO,
we believe that a secondary market for gZIL may open up that will help with the
price discovery of gZIL.

# Limitations and Future Work


# Backward Compatibility

The staking mechanism is intended to work alongside the existing core protocol
and should be fully backward compatible with all its components, i.e., no
change in behavior should be observable on the mainnet operation with this
mechanism in place.

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

