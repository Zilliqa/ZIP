|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 9  | Recycling transactions fees by sending to mining rewards pool | Draft | Standards Track  | Han Wen Chua <hanwen@zilliqa.com>| 2020-06-27 | 2020-06-27

## Abstract

ZIP-9 defines a mechanism to capture the network's growth by means of "burning" the transaction fees instead of rewarding them directly to miners.

## Motivation

Measuring the growth of a blockchain network is difficult when there is no explicit token sink mechanism in place to capture the network's growth. Hence, this ZIP proposes a simple mechanism to measure network activity by opting to "burn" the transaction fees, by means of sending them to the 0x0 address (mining reward pool), instead of rewarding them directly to the miners. This will help ensure that the value of $ZIL is tied directly to its fee market demand, and also ensure that the annual inflation of $ZIL as result of mining block rewards is sustainable in the long run.

If this is implemented, the goal of the network will be to achieve the following:

```
Cumulative fees for DS epoch ÷ DS block rewards = 1
```

This will result in a net annual inflation rate of **0%**, and the mining reward pool in 0x0 address will never get depleted as it is perpetually refilled by the transaction fees. However, this implementation presents a new problem. Miners will not be incentivised to include transactions in a TX block, instead they could just mine empty blocks to earn the block rewards. Therefore, there will need to be a proper mechanism to tie the DS block rewards to the percentage of filled blocks. The proposal is do the following:

```javascript
// At the start of DS epoch, set COINBASE_REWARD_PER_DS = BASE_COINBASE_REWARD_PER_DS
// For each TX block in this DS epoch, do the following:
gasDifference = gasUsed - (gasLimit * GAS_CONGESTION_PERCENT) / 100;
if (gasDifference >= 0) {
  COINBASE_REWARD_PER_DS += gasDifference * GAS_PRICE_MIN_VALUE;
} else {
  COINBASE_REWARD_PER_DS -= gasDifference * GAS_PRICE_MIN_VALUE;
  if (COINBASE_REWARD_PER_DS < BASE_COINBASE_REWARD_PER_DS) {
    COINBASE_REWARD_PER_DS = BASE_COINBASE_REWARD_PER_DS;
  }
}
```

If the block is congested, the DS block reward for this DS epoch will increase, while the opposite is true. However, there is a safeguard (`BASE_COINBASE_REWARD_PER_DS`) for miners to prevent them from losing all rewards in the possible case of no transactions in the network at that particular DS epoch.

In this manner, miners are always incentivised to include as many transactions as possible in the network in order to earn all possible mining rewards, thereby eliminating the issue presented above.

## Specification

### Parameters

- `BASE_COINBASE_REWARD_PER_DS`: 275000
- `DS_MICROBLOCK_GAS_LIMIT` = 1000000
- `SHARD_MICROBLOCK_GAS_LIMIT` = 500000
- `GAS_CONGESTION_PERCENT`: 55
- `GAS_PRICE_MIN_VALUE`: 0.002

### Proposal

In order for the fees and inflation per DS epoch to reach parity, we need to tune some of the current blockchain parameters to make this possible. First, we have to make sure that the `BASE_COINBASE_REWARD_PER_DS` must be equal to all fees allocated to 55% (`GAS_CONGESTION_PERCENT`) of the gasLimit for that DS epoch.

Therefore, we have to adjust the `GAS_PRICE_MIN_VALUE` using the formula below to derive a logical value:

```javascript
// Since gasLimit = DS_MICROBLOCK_GAS_LIMIT + 3 * SHARD_MICROBLOCK_GAS_LIMIT = 2,500,000
BASE_COINBASE_REWARD_PER_DS = GAS_CONGESTION_PERCENT/100 * gasLimit * TX blocks in DS epoch * gasMinPrice
			    = 0.55 * 2500000 * 99 * gasMinPrice
			    = 1237500000 * gasMinPrice
```

If we set a target `gasMinPrice = 0.002 ZIL`, we get:

```javascript
BASE_COINBASE_REWARD_PER_DS = 13750000 * 0.002 = 275000;
```

Without the fees being "burned" due to no transactions on the Zilliqa blockchain, the proposed coinbase rewards will translate to roughly ~12.2% annual inflation relative to current circulating supply. If we take into account of miner:staker allocation of 6:4 for coinbase rewards, and disregard the staker allocation as they are considered "locked" supply (meaning non-circulating), the actual inflation is 7.32%. Last but not least, as the network still has 720 guard nodes (520 DS guard + 200 shard guards) that are not qualifed for mining rewards, hence decreasing the actual inflation lower to 5.124%. This is a still a manageable amount considering no transactions happenning to the chain at any moment, but of course that is not we want to see happen.

### Simulation

Taking into account of a ZRC-2 compliant Fungible-token `Transfer()` transition, assuming the gas used of each ZRC-2 transfer is `~228` and we have `gasLimit = 2500000` per TX block, it will be mean a TX block to handle up to ~10,964 ZRC-2 transfers within each TX epoch. (or ~255 TPS of ZRC-2 transfers transactions at the current block time of 43 seconds)

Achieving this rate of ZRC-2 transfers will not be difficult to do assuming the mass adoption of Zilliqa for micro-payments in advertising or social rewards use cases. Therefore, in this simulation, the target of a healthy network will be to hit ~255 TPS (of ZRC-2 token transfers transactions) constantly or even exceed it in order to hit the net annual inflation of <= 0%.

## Implementation

TBD

## Backward Compatibility

TBD

## Appendix

TBD

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
