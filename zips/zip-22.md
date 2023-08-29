|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 22  | Hybrid Consensus | Draft | Standards Track  | Zoltan Fazekas <zoltan@zilliqa.com> | 2023-08-04 | 2023-08-04


## Abstract

This ZIP proposes mixed validator committees formed from mining nodes and staked seed nodes (SSNs) to increase the decentralization of the pBFT consensus.


## Motivation

Currently, the pBFT consensus is run by mining nodes, most of which participate in mining pools with various hashrates. However, the transition of Ethereum from PoW to PoS in September 2022 caused massive changes in the Ethash mining landscape. If mining pools become more concentrated, it is possible that the fraction of the hashrate controlled by a single pool might grow to close to the 67% level which would pose an issue for the pBFT consensus.

Zilliqa 2.0 will be a Proof of Stake (PoS) network. For a smooth transition from the current network to Zilliqa 2.0, we need a PoS validator community. SSNs are predestined to become the first PoS validators as their accumulated delegated stake already amounts to approx. 33% of the current ZIL supply. To establish themselves as future PoS validators and to help solve the centralization issue described above, SSNs shall be enable to join the DS committee.

## Specification

The current proposal presumes desharding described in ZIP-23 [1] to be implemented simultaneously, so that the hybrid consensus considers only a single DS committee.

The proposal requires changes in the following three areas.

### Validator Selection

The rules for inclusion of mining nodes based on their PoW submissions remains unchanged. With the current DS difficulty and total network hashrate, around 10 new mining nodes join the committee every epoch, replacing other mining nodes that have been part of the committee for the longest time. Additionally, we invite all active SSNs (currently 23 according to [2]) to join the DS committee as long as they fulfill the minimum stake requirement.

### Consensus Rules

Currently, the pBFT commit rule requires more than 2/3 of the committee members to agree on a proposed block. This rule will be extended with a second condition (stake-weighted voting). In addition to the first condition, the second condition requires that more than 2/3 of the stake held by the committe must agree on the same block.

To implement this, the consensus must take a snapshot of the validators' stake from the SSN staking contract during the vacuous epoch. During the TX epochs, it must add up the stake of the validators that co-signed a block and check if the result is larger than 2/3 of the stake included in the snapshot.

### Staking Rewards

Initially, participating in the consensus will be optional. If SSNs opt out, they will continue to receive at most 50% of the staking reward based on the current reward distribution mechanism. In order to earn up to 100% of the staking reward, they must participate in the consensus. At a later stage, participation may become mandatory by reducing the staking reward of non-participating SSNs to zero.

Staking rewards of participating SSNs will not be issued based on their availability tested by the verifier node, but based on their active contribution to the pBFT consensus. The amount of staking reward they receive will be determined by the number of blocks in the DS epoch they co-sign.

Mining rewards remain unaffected by this proposal. 


## Security

The current PoW-based validator selection relies on the adversarial assumption formulated in the Zilliqa whitepaper [3]: "*We assume that the mining network at any point of time has a fraction of byzantine nodes/identities with a total computational power that is at most* $f<\frac{n}{4}$ *of the complete network, where* $0 ≤ f < 1$ *and* $n$ *is the total size of the network.*" Furthermore, "*one can show that if the DS committee size is sufficiently large (say above 800), then among the* $n_0$ *members of the committee at most* $\frac{1}{3}$ *are byzantine with high probability.*"

We can apply a similar argument in the PoS setting by assuming that byzantine nodes control at most $\tilde{f}<\frac{\tilde{n}}{4}$ fraction of the total stake $\tilde{n}$, where $0 ≤ \tilde{f} < 1$. Analogously to the requirement regarding the DS committee size in the whitepaper, it is easy to see that if the committee's stake size is sufficiently large (above the $\frac{3\tilde{n}}{4}$ threshold), then the fraction of the committee's stake controlled by byzantine members of the committee is less than $\frac{1}{3}$. In order to be safe as per the formulated adversarial assumption, we aim for more than 75% of the total stake delegated to SSNs to participate in the consensus and incentivize the SSNs accordingly.


## Future Work

Punishment of validators that deviate from the protocol is a crucial part of the security model of PoS systems. In order to impose penalties, however, honest participants must identify the offenses and submit proofs, and the consensus leader must include them in the proposed block, so that ideally both the block proposer and the whistleblower receive a reward. Due to the additional complexity of these features we defer them to future work on Zilliqa 2.0.


## Conclusion

The hybrid consensus eliminates the dominance of the largest mining pools' nodes in the DS committee as neither of them controls a considerable fraction of the stake. Therefore, the current proposal significantly improves the Zilliqa network's decentralization. By contributing to the network's security, SSNs participating in the hybrid consensus will continue earning staking rewards and put themselves in the position to become the future validators of Zilliqa 2.0.


## References

1. [https://github.com/Zilliqa/ZIP/blob/master/zips/zip-23.md](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-23.md)
1. [https://viewblock.io/zilliqa/staking](https://viewblock.io/zilliqa/staking)
1. [https://docs.zilliqa.com/whitepaper.pdf](https://docs.zilliqa.com/whitepaper.pdf)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
