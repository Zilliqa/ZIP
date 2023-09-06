|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 23  | Desharding | Draft | Standards Track  | Zoltan Fazekas <zoltan@zilliqa.com> | 2023-08-04 | 2023-08-04


## Abstract

This ZIP proposes a change to Zilliqa's sharding structure to reduce the block interval and the operational costs of the network. The proposal also includes a fairer distribution of rewards among miners.


## Motivation

The Zilliqa network uses 4 shards to process transactions in parallel. While 3 of them only deal with simple ZIL transfers, the DS shard alone is responsible for transactions that involve smart contract calls. In each TX epoch, the DS committee waits for the 3 shards to each deliver a microblock before agreeing on the final TX block. This makes the overall block time to take around 30 seconds. The current proposal aims at reducing the block time to 15-20 seconds which is a substantial performance improvement.  

The operation of the 3 shards requires up to 1680 mining nodes to join the network in every DS epoch. The current proposal eliminates the need for this high number of nodes and the mining costs related to their selection, thereby significantly reducing the overall operational costs and carbon footprint of the network.

The mining rewards consist of base rewards and co-signer rewards. While all mining nodes receive the same share of the base reward, only the mining nodes that are among the first ⅔ to provide their signature on the proposed block to the leader receive an equal share of the co-signer reward. This creates a significant disparity between fast and slow mining pools, which the current proposal alleviates to an extent that we believe is acceptable for both.


## Specification

We propose to suspend the operation of the 3 non-DS shards until the launch of new shards based on the Zilliqa 2.0 architecture. The implementation of this proposal requires the following changes to the current Zilliqa protocol.

### Omitting Shard PoW Submissions

The Zilliqa miners solve two PoW puzzles per DS epoch. While the DS PoW is required to determine new members of the DS committee, the shard PoW is used to form the 3 shards. The first up to 1680 nodes that manage to solve the shard PoW puzzle are assigned randomly to one of the 3 shards so that all shards end up with approx. the same number of nodes. The PoW solutions must be submitted within a 60 second window right after the DS submissions. As desharding eliminates the 3 shards, there will be no need for shard PoW in the future.

The DS PoW remains unchanged. To participate in the DS committee, mining pools must solve a puzzle with a difficulty two orders of magnitude higher than the shard difficulty. Because of this, there are only a few new mining nodes in every DS epoch that join the DS committee and replace other mining nodes that have been part of the committee for the longest time. It's obvious that the frequency of solving the more difficult DS PoW puzzle is lower, but at the same time, the reward of the DS PoW winners will be much higher in the future as explained in the section after next. Since the total amount distributed over a period remains unchanged, rewards the mining pools will earn in proportion to their hashrate remain in the long run the same as before.

An analogy to explain this effect can be taken from the Ethereum Mainnet's incentive layer [1], where all validators participate in the consensus in every epoch, but the few block proposers among them are selected randomly. While each validator receives a small reward in every epoch, it only gets the chance to earn the larger proposer reward once in every 83 days on average. So being a validator brings small but constant rewards while proposing blocks gives seldom but large rewards, just as smaller Zilliqa mining pools have their share of nodes in the 3 shards constantly and become DS committee members irregularly.

### Omitting Shard Microblocks

Currently, transactions are assigned to the 3 shards and the DS committee by a lookup node, which knows the composition of the shards. Each shard runs its own instance of the pBFT consensus to agree on a microblock of transactions that were assigned to it. The 3 microblocks are then sent to the DS committee which merges them into a final TX block that also contains transactions that were sent to the DS committee. Finally, the DS committee runs another instance of the pBFT consensus to agree on the TX block.

A variable called `m_sendAllToDS` in the code of the lookup node determines whether it sends all transactions to the DS committee or distributes them across the shards. There are also several constants defined in `constants.xml` that determine the length of the window for various steps such as the dissemination of transaction packets, the block creation and the pBFT consensus itself. These values must be adjusted and tested carefully to avoid destabilization of the DS consensus. Nevertheless, without the 3 shards there will be no need for microblocks and additional consensus runs to agree on them. The DS leader can create the final TX block right away and after all DS committee members received all transactions within a sufficient window, they can run a single instance of the consensus to agree on it.

### Adjusting Mining Rewards

In order to maintain the same total amount of mining rewards as before desharding, we must multiply the rewards distributed among the DS committee members by the constant factor `2.78` established based on the average number of nodes that earned co-signer rewards before desharding.

The reward adjustment does not change the fact that a fast node can earn up to 4 times more than a slow node during the same DS epoch. To alleviate this disparity to some extent, we multiply the co-signer reward by `0.6` and the base reward by `1.7`. The total amount of mining rewards remains unchanged and all of the pools will continue to earn at least 94% of their current rewards. At the same time, the maximum gap between fast and slow pools gets reduced from 4x to approx. 2x.


## Future Work

The sharded architecture of Zilliqa will be not only restored but largely improved in Zilliqa 2.0 with several benefits such as shorter block intervals and faster finality, smart contract support on all shards, cross-shard transactions and many more.


## Conclusion

Desharding will significantly improve the performance of the Zilliqa network by reducing the block time by up to 50%. At the same time, it will save substantial costs in the area of mining as well as node operation. Last but not least, it will create a fairer opportunity to earn rewards regardless of where miners operate their nodes, while continuing to encourage them to deliver signatures as quickly as possible to ensure a smooth operation of the pBFT consensus.


## References

1. [https://eth2book.info/capella/part2/incentives/rewards/](https://eth2book.info/capella/part2/incentives/rewards/) 

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
