| ZIP | Title                        | Status | Type  | Author                                                                                                                       | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) |
| --- | ---------------------------- | ------ | ----- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------- | -------------------- |
| 4   | GetMinerNodes API | Implemented  | Standards Track | Antonio Nunez <antonio@zilliqa.com> | 2020-02-12           | 2020-06-08           |

## Abstract

ZIP-4 provides a way to store and retrieve the miner nodes that comprised the Zilliqa network at a particular epoch.

## Motivation

Since coinbase rewards are not recorded as transactions, miner addresses cannot be ascertained by analyzing the transactions on the chain. While other indirect sources can be used to track miner inflow and outflow activity, the lack of a direct means to obtain miner addresses still makes accurate data analytics a difficult exercise.

In the absence of recorded transactions, the next recourse is to retrieve the network membership dating back to the genesis DS block. However, retrievable information is sparse and limited to the following:

**DS Committee**
- `GetDSBlock` - Returns the new Directory Service committee members (i.e., PoW winners) at the DS block's epoch

**Shards**
- `GetBlockchainInfo` - Returns the shard sizes at the current epoch
- `GetShardMembers` - Returns a random subset of miners in a shard at the current epoch

Thus:

- Identifying the DS committee members at epoch N is impossible even through reconstruction from the genesis DS block. This is because the committee membership is decided by the [DS reputation algorithm](https://github.com/Zilliqa/Zilliqa/pull/1587), the inputs of which are not readily available through the API even though they may be stored on the chain (e.g., Tx block cosignatures).
- Identifying the shard committee members at epoch N is also impossible because the information is not stored on the chain.

## Specification

In Lookup and Seed nodes, a database named `minerInfo` stores the following information at the generation of every DS block:

**DS Committee**
- If current DS block number is a multiple of `STORE_DS_COMMITTEE_INTERVAL`
  - Public keys of the DS nodes (excluding DS guards)
- Otherwise
  - Public keys of the DS nodes ejected from the committee

**Shards**
- For each shard, the shard size and public keys of the shard nodes (excluding shard guards)

The DS block number is used as the database key.

A JSON-RPC API named `GetMinerNodes` accepts a DS block number as input:

```
curl -d '{
    "id": "1",
    "jsonrpc": "2.0",
    "method": "GetMinerNodes",
    "params": ["123"]
}' -H "Content-Type: application/json" -X POST "https://api.zilliqa.com/"
```

and provides the public keys in its response:

```
{
  "id": "1",
  "jsonrpc": "2.0",
  "result": {
    "DSCommittee": [
      "0x0208EA17FE78DE1D4A17B921B7ADB2B3FEE4C795FC5FD54BEA80FCE926F09F79F3",
      "0x0307BA09F607D36C6D3B2AC7541592DB8674A889690A14B816FACF67B4F9221CEC",
      ...
    ],
    "Shards": [
      {
        "Size": 600,
        "Nodes": [
          "0x0342081E95907022E1E5A525F2D7CE51EB5AF56B7EB13C29C6EC194FB93DF2E752",
          ...
        ]
      },
      {
        "Size": 600,
        "Nodes": [
          "0x020C11D6B9CA45B56EC80705F1070D9928CFA4BA99212BEF934307C228ECC309FC",
          ...
        ]
      },
      {
        "Size": 600,
        "Nodes": [
          "0x02CB5328E79387F337EC671ECFF0CF2501CCD4DDFCA3565640F94792DE0A7EA0F9",
          ...
        ]
      }
    ]
  }
}
```

The algorithm behind the `GetMinerNodes` API performs these steps:

**DS Committee**
1. Retrieve the `minerInfo` database entry for the nearest multiple of `STORE_DS_COMMITTEE_INTERVAL`. For example, if `STORE_DS_COMMITTEE_INTERVAL = 10` and the requested DS block is 23, then navigate to the entry for DS block 20.
2. Set the initial DS committee result to the public keys in the entry.
3. From the entry after that until the requested block (i.e., from 21 to 23 in this example):
   1. Retrieve the `dsBlocks` database entry for the current block number.
   2. Add the public keys of the `PoWWinners` in that entry to the DS committee.
   3. Retrieve the `minerInfo` database entry for the current block number.
   4. Remove the public keys of the ejected nodes in that entry from the DS committee.
4. Record the final DS committee public keys in the API response message.

**Shards**
1. Retrieve the `minerInfo` database entry for the requested DS block.
2. Record the shard sizes and public keys in the API response message.

## Rationale

### Node types

This ZIP intentionally excludes other node types (such as Lookup and Seed nodes) since those are relatively fixed and do not receive coinbase rewards as a result of Proof-of-Work mining and block generation. DS and shard guards are excluded for the same reason.

### Keys vs Addresses

Public keys (33 bytes) are longer than Zilliqa `base16` addresses (20 bytes) and thus require more storage space. A public key, however, can be used to derive the address. Additionally, it is common practice across the core protocol databases to store keys instead of addresses. Keys are therefore used in this ZIP due to flexibility and consistency, even though they consume more space.

### Required Storage

Storing the shard public keys in full cannot be avoided due to the re-composition of the shards at every DS epoch.

The DS committee, however, mostly stays intact across adjacent DS epochs, with only a maximum of `NUM_DS_ELECTION` nodes replaced in each epoch. Theoretically, the DS committee can be reconstructed in full without any new storage by re-running the [DS reputation algorithm](https://github.com/Zilliqa/Zilliqa/pull/1587) from the genesis DS block until the requested DS block. Obviously this is computationally expensive and places significant stress on the Lookup and Seed nodes.

By storing the full list of DS committee members (excluding guards) at regular intervals indicated by `STORE_DS_COMMITTEE_INTERVAL`, and limiting the reconstruction of members to between two of those intervals, a balance between storage and computation requirements is achieved.

## Backward Compatibility

This feature is not backward compatible and thus cannot be applied to the blockchain before the point of implementation. That means a call to `GetMinerNodes` for a DS block before this feature is implemented returns an error message.

## References

- [API documentation](https://apidocs.zilliqa.com/)
- [DS reputation algorithm](https://github.com/Zilliqa/Zilliqa/pull/1587)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).