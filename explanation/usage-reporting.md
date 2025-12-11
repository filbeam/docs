---
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Usage Reporting

This document explains how FilBeam reports usage on-chain and what this means for transparency and verification.

For the complete payment flow, see [Payment Model](payment-model.md).

## Why On-Chain Reporting?

FilBeam records all egress usage on the Filecoin blockchain to ensure transparency, enabling verification to storage providers, and support for on-chain settlement. All reported usage data is publicly visible on-chain, allowing storage providers to independently verify that reported usage matches their delivery logs. 

This design also lets storage providers settle their payment rails at any time without relying on FilBeam to initiate or distribute payments.

## What Gets Reported

FilBeam reports two metrics for each data set:

| Metric | Description | Purpose |
|--------|-------------|---------|
| **CDN bytes** | Total egress (cache hits + cache misses) | Determines FilBeam's CDN fees |
| **Cache-miss bytes** | Only traffic fetched from storage providers | Determines storage provider compensation |

### What's Excluded

Not all traffic counts toward billing:

- **Invalid responses** - Cache misses where the response was invalid are not counted towards total bytes used
- **Empty responses** - Only responses with actual bytes delivered are counted towards total bytes used

## Important Caveat

**Only traffic proxied through FilBeam is reported on-chain.**

If users retrieve content directly from storage providers (bypassing FilBeam):
- That traffic is NOT recorded in FilBeam's systems
- It is NOT reported to the blockchain
- It is NOT subject to FilBeam billing
- The Storage Provider is NOT rewarded for the retrieval service


This is by design - FilBeam only bills for traffic it actually serves.

## Reporting Schedule

Usage is aggregated and reported periodically:

| Network | Frequency |
|---------|-----------|
| Calibration | Every 30 minutes |
| Mainnet | Every 4 hours |

Usage is reported up to the previous complete Filecoin epoch, ensuring only finalized data is recorded on-chain.

## Verification and Trust Model

Storage providers can verify reported cache-miss bytes by:

1. Monitoring `recordUsageRollups` transactions on the FilBeamOperator contract
2. Comparing reported bytes with their own delivery logs for requests served via FilBeam

Earnings are calculated as: `cache_miss_bytes × $7/TiB`

Users storing their content can view reported usage on-chain but have no mechanism to independently verify that reported bytes match their actual usage.

## See Also

**Explanations:**
- [Payment Model](payment-model.md) - Complete payment flow from top-up to settlement
- [Quota System](quota-system.md) - Understanding the dual quota design

**How-To Guides:**
- [Observe Reported Usage](../how-to/observe-reported-usage.md) - Monitor on-chain usage data

**Reference:**
- [FilBeamOperator Contract](../reference/filbeam-operator.md) - Complete ABI documentation
