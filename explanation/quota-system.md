# Quota System

This document explains why FilBeam uses a dual quota system and how it affects your content delivery.

For the complete payment flow including top-ups, usage reporting, and settlement, see [Payment Model](payment-model.md).

## What is a Quota?

A quota represents the amount of data you can transfer through FilBeam, measured in bytes. When you deposit funds on-chain, FilBeam derives your quota off-chain based on the deposited amount and the current price per byte.

As content is delivered, your quota is consumed. When quota runs out, requests are blocked until you top up.

## Why Dual Quotas?

FilBeam uses two separate quotas to enable **direct payment rails** between you and storage providers:

1. **CDN Egress Quota** - Derived from funds you deposit into a payment rail between you and FilBeam.
2. **Cache Miss Egress Quota** - Derived from funds you deposit into a separate payment rail between you and the storage provider.

The key design principle is that FilBeam never handles storage provider funds. When you top up cache-miss quota, those funds go into a payment rail that only you and your storage provider can access—FilBeam simply tracks and reports usage. This creates a direct financial relationship between you and your storage provider.

## Quota Exhaustion

### When CDN Quota Runs Out

All requests are blocked with HTTP 402, regardless of cache-miss quota remaining.

**Why?** Every request goes through FilBeam's CDN, so CDN quota is always required.

### When Cache-Miss Quota Runs Out

- **Cache hits continue working** - cached content is still served
- **Cache misses are blocked** with HTTP 402

**Why?** You can still serve content that's already cached, but new content cannot be fetched from storage providers until you top up.

This design lets you continue serving popular content even if you've exhausted your cache-miss budget.

## Choosing a Top-Up Strategy

How you split your top-up between CDN and cache-miss quotas depends on your content access patterns:

### Frequently Accessed Content

If your content is accessed repeatedly (high cache hit ratio):
- **Favor CDN quota** - most requests will be cache hits
- Example split: 90% CDN, 10% cache-miss

### Infrequently Accessed Content

If your content is accessed infrequently (low cache hit ratio):
- **Balance both quotas** - more requests will be cache misses
- Example split: 50% CDN, 50% cache-miss

### Unknown Access Patterns

Start with a balanced split and monitor your actual cache hit ratio via the Stats API. Adjust future top-ups based on observed patterns.

## FAQ

### Why do I need both quotas?

The dual quota system enables direct payment rails between you and each service provider. CDN quota pays FilBeam for content delivery. Cache-miss quota pays your storage provider directly for content retrieval. FilBeam never receives or distributes storage provider funds which keeps the payment flow simple and transparent.

### What if I run out of cache-miss quota but have CDN quota?

Cached content continues to be served normally. Only requests that would require fetching from the storage provider are blocked. This lets you keep serving cached content while you top up.

### Can I convert one quota type to another?

No. Each quota type is purchased separately and serves a different purpose. Top up the specific quota you need based on your observed usage patterns.

### How do I check my current quotas?

See [Monitor Usage](../how-to/monitor-usage.md) for instructions on checking your quota balances.

## See Also

**Explanations:**
- [Payment Model](payment-model.md) - Complete payment flow from top-up to settlement
- [Usage Reporting](usage-reporting.md) - How usage is reported on-chain

**How-To Guides:**
- [Top Up CDN Quota](../how-to/top-up-cdn-quota.md) - Step-by-step instructions
- [Monitor Usage](../how-to/monitor-usage.md) - Check your quotas and usage
