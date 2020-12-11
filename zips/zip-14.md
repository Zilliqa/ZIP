|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 14  | Revised pBFT consensus | Draft | Standards Track  | Sandip Bhoir <sandip@zilliqa.com>| 2020-12-10 | 2020-12-10

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

![new_flow](../assets/zip-14/new_flow.png)

These are important points to consider:

**Round 1:**
- Round 1 of consensus also called preprep phase will run consensus on microblock/finalblock with txnhashes only.
- Leader starts actual transaction processing after sending out preprep announcement ( mb/fb with txnhashes only).
- Backups starts actual transaction processing after sending out commit for preprep phase.
- However, if there is commit failure ( ex. due to missing txns), it will fetch the missing ones from leader and rerun preprep phase (round-1) of consensus.
- For any other commit failure, please refer the new flow.

**Round 2:**
- Leader after processing all transaction or after timeout is hit, sends collective sig  + mb/fb with processed transactions to all backups.
- Backup is expected to complete processing of transaction locally that were proposed in preprep phase by now.
- Backup validates and send final commit.
- However, if there is commit failure, it will move to ERROR state and proceed as earlier design.
 
Note: This revised pBFT is applicable only for shard microblock consensus and final block consensus which basically involves transactions.

## Implementation

This ZIP is implemented in the following pull requests in the Zilliqa core repository:
- [PR 22216](https://github.com/Zilliqa/Zilliqa/pull/2216)


## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).