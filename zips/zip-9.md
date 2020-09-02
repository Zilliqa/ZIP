| ZIP | Title                       | Status | Type            | Author                            | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) |
| --- | --------------------------- | ------ | --------------- | --------------------------------- | -------------------- | -------------------- |
| 9   | Tokenomic changes for \$ZIL | Draft  | Standards Track | Han Wen Chua <hanwen@zilliqa.com> | 2020-06-27           | 2020-08-02           |

## Abstract

ZIP-9 defines the details for the new token economics (OR tokenomics) changes for $ZIL. The purpose of this exercise is to restructure the block reward distribution in order to accommodate uncapped staking in the seed node staking program, while retaining sufficient incentives for the miners/validators to continue propagating the network. This proposal also introduces a mechanism to accrue value to $ZIL as the network's economy grow and transactions on the network increases.

## Motivation

Staking on Zilliqa is meant to create a symbiotic relationship between the seed node operators and the $ZIL token holders. The token holders gain staking rewards for contributing their capital (in $ZIL), and the seed node operators gain commission for operating seed nodes and staking delegated tokens on behalf of the token holders. However, there is a restriction in the block rewards allocated to the seed node staking program right now, resulting in limitations in the the implementation of the staking program. Not all token holders get to participate due to the cap on maximum amount of tokens that could be staked at any one point of time, and the seed node operators have to charge high fees or risk losing money from the operational cost of hosting the seed nodes. Therefore, there is a need to restructure the block reward distribution in order to create a better version of the staking program, allowing for an uncapped staking pool and sufficient staking interest rate that can attract token holders to participate and stake their tokens in the pool. This will be done by both increasing the block rewards and reallocating some of the miner's share of the block rewards to the stakers.

However, with the changes above, the network will experience high inflation if there is no token sink mechanism to capture the annual increase in supply. Therefore, we need an elegant token sink mechanism in order to ensure that inflation does not dilute the token value, while allowing for the the token to accrue value due to the growth in the network's transactions activity. Hence, this ZIP also proposes a simple token sink mechanism by opting to "burn" the transaction fees, by means of sending them to the 0x0 address (mining reward pool), instead of rewarding them directly to the miners and adding to the circulating supply. This will help ensure that the value of $ZIL is tied directly to its fee market demand, and also ensure that the annual emission of block rewards is sustainable in the long run as the maximum supply of $ZIL is capped at 21 billion. If the network's transactions activity is to grow, the network will be able to achieve low net inflation or even net deflation of its circulating supply.

## Specification

### Changes in Parameters

#### Zilliqa Core

- `BASE_COINBASE_REWARD_PER_DS` = 275000
- `LOOKUP_REWARD_IN_PERCENT` = 40
- `BASE_REWARD_IN_PERCENT` = 20 (Overall share increase from 26.31% to 33.33% for the miner allocation)
- `GAS_PRICE_MIN_VALUE` = 0.002
- Gas fees burn = 100%

#### Scilla Interpreter

- `Gas.scale_factor` = 8

### Details of Proposal

With the changes in the parameters above and implementation of the "burning" of transaction fees, we can simulate net inflation against the average filled block as shown below:

| Average filled block (%) | Net inflation (%) |
| ------------------------ | ----------------- |
| 10                       | 9.99              |
| 20                       | 7.80              |
| 30                       | 5.60              |
| 40                       | 3.41              |
| 50                       | 1.22              |
| 60                       | -0.97             |
| 70                       | -3.17             |
| 80                       | -5.36             |

If we are to take into account of the conservation usage of the network at an average 50% filled block (or ~120 ZRC2 token transfers per second), we can also simulate the YoY growth of circulating supply and its corresponding net inflation:

| Year | Circulating supply | Net inflation |
| ---- | ------------------ | ------------- |
| 0    | 13,595,083,582.93  | 1.22          |
| 1    | 13,760,702,332.93  | 1.20          |
| 2    | 13,926,321,082.93  | 1.19          |
| 3    | 14,091,939,832.93  | 1.18          |
| 4    | 14,257,558,582.93  | 1.16          |
| 5    | 14,423,177,332.93  | 1.15          |
| 6    | 14,588,796,082.93  | 1.14          |
| 7    | 14,754,414,832.93  | 1.12          |
| 8    | 14,920,033,582.93  | 1.11          |
| 9    | 15,085,652,332.93  | 1.10          |

Therefore, net inflation of the circulating supply will be kept low and decreases over the years. And, we can see that the circulating supply at the 10-year mark will still be kept within the 21 billion maximum cap supply limit set out in the white paper. Therefore, allowing us to achieve low net inflation while keeping \$ZIL scarce.

Achieving the above will make sure that we can offer sufficient staking rewards for the token holders while not massively diluting the circulating supply, simulation of the staking rewards against the circulating supply staked can be seen below:

| Circulating supply staked (%) | Staking interest rate (%) |
| ----------------------------- | ------------------------- |
| 10                            | 48.73                     |
| 20                            | 24.36                     |
| 30                            | 16.25                     |
| 40                            | 12.18                     |
| 50                            | 9.75                      |
| 60                            | 8.12                      |
| 70                            | 6.96                      |
| 80                            | 6.09                      |
| 90                            | 5.41                      |
| 100                           | 4.87                      |

As shown above, even at 100% circulating supply staked, we can support up to 4.87% staking interest rate for the token holders. Therefore, proving that we can have an uncapped staking program while retaining sufficient interest rate to encourage participation from token holders. Also, if we are to be moderately optimistic of the participation in the seed node staking program, and take into account of a fixed 80% of circulating supply being staked at any point of time, we can simulate the YoY changes of interest rate with regards to YoY growth of the circulating supply shown above:

| Year | Staking interest rate (%) |
| ---- | ------------------------- |
| 0    | 6.091                     |
| 1    | 6.018                     |
| 2    | 5.946                     |
| 3    | 5.876                     |
| 4    | 5.808                     |
| 5    | 5.741                     |
| 6    | 5.676                     |
| 7    | 5.613                     |
| 8    | 5.550                     |
| 9    | 5.489                     |

Therefore, we can see that the staking interest rate over the years will still be sufficient to attract the participation of token holder in the long run, and allow token holders to combat the projected annual net inflation shown above earlier. As a result, the token holders \$ZIL holdings when staked will actually appreciate in value over time.

To conclude, these proposed parameters changes and implementation of "burning" of transaction fees will allow for a better staking program that encourages token holders participation, and also allow the network to keep net inflation low so to not dilute the value of the token.

## Implementation

TBD

## Backward Compatibility

TBD

## Appendix

TBD

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
