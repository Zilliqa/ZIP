
|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 18  | Simple Transaction Spamming Mitigation | Ready | Standards Track  | Julian Sarcher <julian@sarcher.de> | 2021-02-08 | 2021-02-08

# Summary

The proposal addresses transactions spamming at the Zilliqa blockchain by increasing the fees of payment transactions, making it more costly for anyone to spam the network. Smart contract transactions, e.g. claiming staking rewards or ZilSwap swapping, are **NOT** effected by this proposal. Additionally, this proposal effects the circular token-economic model of Zilliqa by potentially increasing the amount of ZILs burned. 

# Abstract

The proposal has the following main effects:


- Minimum payment transaction cost will be increased from **0.002 ZIL** to **0.1 ZIL** if this proposal gets implemented
- Smart-contract execution fees in ZIL **remain as is** if this proposal gets implemented
- Due to increased fees for normal transactions, the inflation rate of ZIL could decrease as more fees will be burned 

Considering the number of transactions executed on the Zilliqa blockchain remains at the same level, more ZILs will be burned which will be beneficial for the circular token-econmic model of Zilliqa by decreasing the inflation rate.

# Motivation

On Feb 4, 2021 on the Zilliqa Blockchain 1,000,000 transactions got executed at the cost of 2000 ZILs of fees [1]. Preventing the blockchain being spammed without touching crucial blockchain attributes like permissionless or trustless, fees for normal ZIL transactions need to be adjusted.

A comparision of ZRC-2 token transfers and ZIL transfers shows the inequalitity of TXs:

- A ZRC-2 token transfer is usually about 1 ZIL in fees.
- A normal ZIL TX costs 0.002 ZILs.

In order to prevent spamming the blockchain with millions of useless transactions, the proposal suggests to increase the fees for normal TXs to 0.1 ZIL. The gap in fees between ZRC-2 token TXs and ZIL TXs is just not comprehensible.

# Specification

The increase of payment transaction fees can be done without effecting the fees associated with smart contract execution by increasing the gas unit consumption of payment transactions:

- Changing the constant value of **NORMAL_TRAN_GAS** from **1** to **50**

Furthermore, the currently extremely low transaction fees make little sense with respect to the token-economic model of Zilliqa [2], as it requires burning fees in order to get ZIL inflation rate at meaningful levels, let’s say < 5%.

- DS Block reward = 275,000 ZIL [3]
- Circulating Supply = 14.2e9 ZIL [4]

Assuming 1 TX block is produced roughly 45 seconds and there are 100 TX Blocks per DS epoch:

- Reward per second = 275,000 ZIL / (45s*100) = ~61.1 ZIL/s
- Base inflation rate = 61.1 ZIL/s x (365 x 24 x 3600) / 14.2e9 = ~13.57%

According to the circular economic model Zilliqans are obliged to spend those minted ZILs by paying fees to the network. Assuming the zero inflation state when fees payed equals the block rewards, with the current network configuration this relates to one of:

**A)** ~61 ZRC-2 token transfers per second (assuming 1 ZIL per TX)

**B)** ~30,550 TX per second (assuming 0.002 ZIL per TX)

**C)** ~15 ZilSwap swaps per second (assuming 4 ZIL per swap)

If the proposal gets implemented only **B)** will change so that ~611 TX per second are required for reaching the zero inflation state of Zilliqa’s token-economic model:

**A)** ~61 ZRC-2 token transfers per second (assuming 1 ZIL per TX)

**B)** ~611 TX per second (assuming 0.1 ZIL per TX)

**C)** ~15 ZilSwap swaps per second (assuming 4 ZIL per swap)

Assuming a TX block time of 45 seconds, then, 611 TX per second relate to 27,495 TX per block. Maximum number of TX per block ever recorded on mainnet was slightly above 10,000 TX per block. Maximum block capacity when filled only with normal transactions would be 50,000 TX, which relates to ~1,111 TX per second. Currently, on the mainnet we have ~0.46 TX per second on a normal day (40,000 TX per day).

**With maturing of the network and increased adoption only the gas price needs to be lowered for higher maximum TPS in the network.**

The rationale for the value 0.1 ZIL is based on the ratio of fees for payment transactions to other blockchain functions like sending a ZRC-2 token (1 ZIL) or swapping on ZilSwap (4 ZIL).

Current ratios:
- ZRC-2 token transfer: 1:500
- Zilswap swap: 1:2000

Proposal’s ratios:
- ZRC-2 token transfer: 1:10
- Zilswap swap: 1:40

It has to be noted that the implemenation of the proposal requires a mandatory network upgrade for all nodes, rather than a optional one. Furthermore, the roll-out would need to be carefully planned by the Zilliqa team, e.g. informing and coordinating exchanges and wallets and updating various SDKs to ensure support.

# For

1. Higher transaction fees will make spamming very costly, e.g. issuing 1,000,000 TX would cost 100,000 ZIL.
2. Higher transaction fees could decrease inflation rate of ZIL.
3. The network is still in its early days, it needs to be protected from useless overload.
4. The ratio of ZRC-2 token TX to Normal TX gets more reasonable.
5. The fee of 0.1 ZIL is still low and gas price can be lowered with maturing of the network in the future.

# Against

1. The transaction fee for a payment transaction increases from 0.002 ZIL to 0.1 ZIL
2. Due to higher fees the number of transactions per day executed on the Zilliqa blockchain could decrease.
3. Centralized exchanges’ withdrawal fees increase from 0.002 ZIL to 0.1 ZIL as well as for exchanges’ internal transactions, e.g. to and from their cold wallet.
4. Risk of failed (rejected) transactions during roll-out of the change.
5. Requires a mandatory network upgrade for all nodes.

# References

1. https://viewblock.io/zilliqa/stat/txCountHistory

2. https://github.com/Zilliqa/ZIP/blob/master/zips/zip-9.md

3. https://viewblock.io/zilliqa/block/1004899

4. https://viewblock.io/zilliqa/stat/topHolders

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
