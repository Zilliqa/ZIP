|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 15  | Early Packet Dispatching for Shards | Draft | Standards Track  | Sandip Bhoir <sandip@zilliqa.com> <br> Haichuan Liu <haichuan@zilliqa.com> | 2020-12-11 | 2020-12-24

## Abstract

This ZIP proposes changes to the timing around the dispatch of transaction packets by lookup nodes to shard and directory service nodes. It also proposes changes to the time when transaction packets will be gossiped and processed within the shards.

## Motivation

Transaction [soft confirmation](https://github.com/Zilliqa/Zilliqa/pull/2154) was introduced in Zilliqa version [7.0.0](https://github.com/Zilliqa/Zilliqa/releases/tag/v7.0.0). Soft confirmation means microblocks are now sent out by each shard to the lookup nodes as soon as microblock consensus is completed, instead of after the final block is generated.

Currently, when a shard has completed both microblock consensus and sending of microblocks to lookup nodes, its shard nodes stay idle waiting for the final block from the DS committee. This idle time could be used to receive transaction packets from lookups, and to gossip those packets within the shard. This change would enable us to shorten the `TX_DISTRIBUTE_TIME_IN_MS` and thereby shorten the block time.

## Specification

The major changes involved are as follows:

| Node   | Current Behavior | Proposed Behavior |
|--------|------------------|-------------------|
| Lookup | Dispatch packets for next epoch to shards after receiving final block | Dispatch packets for next epoch to a shard after receiving its microblock |
|        | Dispatch packets for next epoch to DS after receiving final block | No change |
| Shard  | Buffer packets if node is in improper state* | No change |
|        | Buffer packets if received from lookup (regardless of state) | Gossip packets received from lookup immediately (as long as not in improper state) |
|        | Gossip previously buffered packets at the start of new epoch | No change |
|        | Process packets at start of new epoch** | No change |
| DS     | 

> (*) Improper state means `m_txn_distribute_window_open && (m_state == MICROBLOCK_CONSENSUS_PREP || m_state == MICROBLOCK_CONSENSUS)` is false.

> (**) Processing packets means reading out its contents and adding its transactions into the node's transaction pool.

Nothing changes for DS nodes with regards to transaction packet gossiping and processing timings.

## Implementation

This ZIP is implemented in the following pull requests in the Zilliqa core repository:
- [PR 22216](https://github.com/Zilliqa/Zilliqa/pull/2216)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
