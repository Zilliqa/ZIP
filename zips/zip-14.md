|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 14  | Revised pBFT consensus | Draft | Standards Track  | Sandip Bhoir <sandip@zilliqa.com> <br> George Pîrlea george@zilliqa.com | 2020-12-10 | 2020-12-24

## Abstract

This ZIP details how **Revised pBFT** is adopted to add sustainability to network in high load circumstances where nodes may have varying transactions set.

## Motivation

In the past, the transactions were processed before each node participates in consensus. If nodes misses receiving any of the transactions, it turns out to be too late for him to request the missing transactions from leader and rerun consensus again.
With new pBFT, we handle above issue and thereby adding sustainability enforcing network to progress reliably.

## Specification

### Old pBFT
Please refer below for more details on old behaviour of pBFT for shard microblock.

![existing_flow](../assets/zip-14/existing_flow.png)

### Revised PBFT
Please refer below for more details on new behaviour of pBFT for shard microblock.
Same applies for finalblock consensus.

![new_flow](../assets/zip-14/revised_new_flow.png )

These are important points to consider:

**Round 1:**
- Round 1 of consensus also known as the `pre-prepare` phase will run consensus on microblock/finalblock with a set of transaction hashes. This phase is for the committee to agree on the set of transactions to process.
- Leader starts transaction execution after sending out `pre-prepare` announcement ( mb/fb with transaction hashes only).
- Backups start transaction execution after sending out commit to the leader for `pre-prepare` phase.
- However, if there is commit failure ( ex. due to missing transaction), the backup will fetch the missing transactions from leader and rerun the `pre-prepare` phase (round-1) of consensus.
- For any other commit failure, please refer the new flow.

**Round 2:**
- Leader after executing all transaction or after timeout is hit, sends collective sig  + mb/fb with processed transactions to all backups.
- Backup is expected to complete processing of transaction locally that were proposed in preprep phase by now.
- Backup validates and send final commit.
- However, if there is commit failure, it will move to ERROR state and proceed as earlier design.
 
Note: This revised pBFT is applicable only for shard microblock consensus and final block consensus which basically involves transactions.

## Implementation

This ZIP is implemented in the following pull requests in the Zilliqa core repository:
- [PR 22216](https://github.com/Zilliqa/Zilliqa/pull/2216)


## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
