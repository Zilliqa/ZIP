|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 15  | Early Packet Dispatching for shards | Draft | Standards Track  | Sandip Bhoir <sandip@zilliqa.com> <br> Haichuan Liu <haichuan@zilliqa.com> | 2020-12-11 | 2020-12-24

## Abstract

This ZIP briefs on time when the transaction packets will be dispatched to shard and ds by lookups.
It also briefs on time when transaction packets will be gossiped and processed within the shard.

## Motivation

In the past, shard nodes after finishing microblock consensus were idle waiting for final block from ds committee. This idle time could be used to receive a transaction packets from lookup and gossiped within shard.
This will enable us to shorten the `TX_DISTRIBUTE_TIME_IN_MS` and thereby shorten the block time.

## Specification

Two major changes involved are as follows:

**Lookup:**

Lookup now dispatches transaction packets for next epoch to shard upon receiving of soft confirmation with microblock from corresponding shard for current epoch.
Lookup continues to dispatch the transaction packets to ds-shard for next epoch on receiving final block of current epoch as before.

**Shard-Node:**

Shard node starts gossip immediately upon receiving of transaction packet from lookup.
- Previously, packets were buffered if either received from lookup or if node is in in-proper state. i.e. if `m_txn_distribute_window_open && (m_state == MICROBLOCK_CONSENSUS_PREP || m_state == MICROBLOCK_CONSENSUS)` holds `false`
- Now, the packet will be buffered **only** if node is in in-proper state i.e. if `m_txn_distribute_window_open && (m_state == MICROBLOCK_CONSENSUS_PREP || m_state == MICROBLOCK_CONSENSUS || m_state == WAITING_ON_FINALBLOCK)` holds `false`. Otherwise, shard node will gossip the packet immediately irrespective of whether received from lookup or from peer.

It holds the packet processing until finalblock for current epoch is received. After which it commits transactions from packet into transaction pool.

**DS-Node:**

Nothing changes for DS node with regards to transaction packet gossiping and processing timings.

## Implementation

This ZIP is implemented in the following pull requests in the Zilliqa core repository:
- [PR 22216](https://github.com/Zilliqa/Zilliqa/pull/2216)


## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).