| ZIP | Title | Status | Type  | Author| Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) | 
| --- | ---------------------------- | ------ | ----- |----------------- | -------------------- |-------------------- | 
| 20  | Adjustment of bonding period for Staking phase 1.1 | Draft  | Standards Track | Jun Hao Tan <junhao@zilliqa.com> | 2021-06-15 | 2021-06-15 |

# Table of Content

- [Summary](#summary)
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Specification](#specification)
  * [Estimation of weekly blocks production rate](#estimation-of-weekly-blocks-production-rate)
  * [Bonding period comparsion](#bonding-period-comparsion)
  * [Parameter proposed to adjust](#parameter-proposed-to-adjust)
- [For](#for)
- [Against](#against)
- [References](#references)
- [Copyright Waiver](#copyright-Waiver)



# Summary

This proposal reduces the bonding period of Zilliqa staking phase 1.1 from ~2 weeks to ~1 week. 

# Abstract

This proposal adjusts the bonding period of Zilliqa staking phase 1.1 from 30,800 blocks to 14,500 blocks.

# Motivation

Since Staking phase 1.0 launched, the bonding period has set to ~2 weeks. Since then, there have been discussions on adjusting the bonding period
of Zilliqa staking phase 1.0/1.1. A recent straw poll with 98 voters (as of the writing of this ZIP) on [Zilliqa governance forum](https://gov.zilliqa.com/t/reduce-the-unstaking-period/568/13?u=junhaotan) indicated that many will like to reduce the bonding period to a period of ~1 week.

# Specification

## Estimation of weekly blocks production rate

To estimate the weekly block production rate, we will get the average of weekly production rate based on the last 3 weeks (since Zilliqa v8).

| Week | Number of blocks produced |
| ---- | ------------------------- |
| Week 21 2021 | 14984 |
| Week 22 2021 | 14378 |
| Week 23 2021 | 14642 |
| Average weekly block production rate | 14668 |

## Bonding period comparsion

| Status | Bonding period | Estimated duration |
| ---- | ------------------------- |
| Current | 30800 blocks | ~2.124 weeks | 
| Proposed | 14500 blocks |  ~1.012 weeks |

## Parameter proposed to adjust

`bnum_req` to be changed to `14500` 

# For

The bonding period will be reduced from 30800 blocks to 14500 blocks. Based on the current average block generation rate, the bonding period is estimated to reduce from
2.124 weeks to 1.012 weeks.

# Against

No change to bonding period

# References

1. https://gov.zilliqa.com/t/reduce-the-unstaking-period/568

# Copyright Waiver

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
