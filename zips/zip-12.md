
|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 12  | Standardisation of ZIP processes | Draft | Standards Track  | Han Wen Chua <hanwen@zilliqa.com>| 2020-12-10 | 2020-12-16

# Summary

The purpose of this proposal is to standardise the Zilliqa Improvement Proposal (ZIP) introduction, voting, and implementation process that governs the Zilliqa protocol.

# TL;DR

1. All proposals must be discussed for at least 3 days with a forum poll before they will be assigned a ZIP number by a ZIP editor and moved to Snapshot voting.
2. The Snapshot vote requires > 30 $gZIL to initiate. It will be binding, and must be open for at least 5 days. At least 20% quorum must be reached, and at least 50% must vote “For” the proposal for it to pass.
3. Most open-source core protocol updates by the Zilliqa core team are not required to go through this voting process; this process only governs those changes or proposals that should be ratified as ZIPs which will be discussed, voted on and officially enacted.

# Abstract

This proposal formalises the process for introducing, voting, and implementing ZIPs. Valid proposals are to be discussed for at least 3 days on Zilliqa’s governance forum[1] and include a forum poll to gauge sentiment. If after at least three days there is a 25% “For” vote in the forum poll it may then move to formal voting via Snapshot. In order for a vote to pass it must have a quorum of > 20% of the token total supply and a majority support of > 50%, after at least five days of voting. Following a successful vote, necessary changes will be implemented by Zilliqa’s protocol team and the network’s upgrade timeline will be announced after the implementation is completed. Changes to this policy, including quorum requirements or what constitutes a majority vote, can only be enacted by a valid ZIP that overwrites this policy.

# Motivation

Although there are several informal standards governing the ZIP introduction, voting, and implementation process there is no single, clear policy. ZIP 0[[2](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-0.md)] outlined this process, but it focused largely on the proposal only after it has transitioned to the voting/ZIP stage, and was enacted when all voting was on-chain. So far, no ZIP or formal policy has been implemented that specifies votes conducted via Snapshot are formal and binding. This ZIP would define and formalise the process in order for proposals to be valid and binding, and reduce any confusion in the ZIP introduction and voting process.

# Specification

#### Introducing the ZIP

In order to submit a potential ZIP for voting a user must first create a thread for the proposal on the Zilliqa governance forum[1]. Complete the auto-populated fields that appear when creating a proposal on the forum. A screenshot of these fields are below:

![Forum post example](https://lh5.googleusercontent.com/UzCYmY1TS2Ixx0QKP825BM_uwvYLxGHCq4jecEcmkbMDDSiprcB6Z_DN0aIp8pVa8iT-Q8mMASQKB4kXpd6ItX3UhY2GsG5qe464wuccEcyvmQoVCDYtsXjAd8yG7p-nZnH0Btlu)

Here are some pointers on how to create a proper proposal:

- Stick to the auto-generated template.
- Write concise title with  **no number**. The number will be added by mods once on-chain voting starts.
- Add understandable aka non-tech aka ELI-5 summary.
- Add an abstract: what  **will**  be done if the ZIP is implemented (keep it below 200 words).
- Write a longer motivation with links and references if necessary.
- Add specifications if necessary.
- Formulate clear  **for**  and  **against**  positions.

Additionally, the thread should include a poll from the governance forum to gauge interest from the wider governance community. This poll must be conducted using Discourse’s native poll. After the thread has been on the governance forum for at least 3 days and has received over 25% “For” votes it can proceed to formal voting via Snapshot. This allows the community to suggest potential changes to the proposal before it moves to the formal voting phase on Snapshot. Please do not assign your proposal a ZIP number; numbers will be assigned by moderators prior to a vote taking place.

If a proposal was introduced on the governance forum and achieved at least a 25% “For” from the poll, but is not submitted to Snapshot within 30 days, the author of the proposal must re-submit the proposal to the governance forum and restart the process. This ensures that proposals that previously received support from governance still retain support from the community.

#### Formal Voting Phase

Snapshot is used for formal, binding votes. The user who authored the ZIP will also create the proposal on Snapshot[3]. Snapshot requires > 30 $gZIL[[4](https://viewblock.io/zilliqa/address/zil14pzuzq6v6pmmmrfjhczywguu0e97djepxt8g3e)] in a user’s wallet to create a proposal. If the author does not meet this requirement then contact a moderator who will submit the proposal on your behalf. Before creating the Snapshot vote, please wait for a moderator to assign your ZIP a number and begin your Snapshot title with it.

A minimum of 120 hours (5 days) for voting, and a 20% quorum requirement is required for ZIPs voting processes. This wide time period for voting, coupled with an active communication of ZIPs via the governance forum and on social media channels should help ensure that no ZIPs will slip through and no malicious ones are approved.

The block selected for the Snapshot vote will be block number at the Snapshot submission. In order for a vote to pass it needs to have a majority approval (>50%) by eligible voters. The eligible vote is defined as gZIL held in the wallets to the token holders at the snapshot block. If the Snapshot vote does not meet a 50% majority approval, then the vote is rejected and no changes will be enacted. Authors of proposals that are rejected may resubmit their proposal, but should include significant changes that address issues that may have prevented the ZIP from passing during the initial vote.

### Scope

This specification aims to clarify which proposals should move to the ZIP stage, how long they should be discussed for, and how long the vote should be open for. By only allowing proposals to move to ZIP/voting after several days of discussion, we ensure that everyone’s voice is heard, and proposals should more accurately reflect community consensus.

Additionally, while some may think that 3 days is too short to adequately discuss a proposal, this is simply the minimum requirement. We expect most discussions to last for significantly longer than this, with only a select few well-planned and researched proposals with near unanimous approval moving through so quickly. This proposal also will not stifle the open source protocol improvements that are made daily by dozens of Zilliqa’s core team members and community contributors. It only aims to govern those proposals that seek community feedback and ratification as ZIPs.

### Other Uses for Snapshot

Snapshot may still be used for informal signal voting, community contests and grantsDAO disbursement, but its primary purpose will be to conduct formal, binding votes for ZIPs.

# References

1. [https://forum.zilliqa.com/](https://forum.zilliqa.com/)
2. [https://github.com/Zilliqa/ZIP/blob/master/zips/zip-0.md](https://github.com/Zilliqa/ZIP/blob/master/zips/zip-0.md)
3. [https://governance.zilliqa.com/](https://governance.zilliqa.com/)
4. [https://viewblock.io/zilliqa/address/zil14pzuzq6v6pmmmrfjhczywguu0e97djepxt8g3e](https://viewblock.io/zilliqa/address/zil14pzuzq6v6pmmmrfjhczywguu0e97djepxt8g3e)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
