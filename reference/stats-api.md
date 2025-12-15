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

# Stats API Reference

The FilBeam Stats API provides endpoints for querying egress quota and usage statistics.

## Base URL

```
https://calibration.stats.filbeam.io
```

## Authentication

The Stats API is publicly accessible and does not require authentication.

## Endpoints

### Get Data Set Stats

Retrieve quota information for a specific data set.

```
GET /data-set/{dataSetId}
```

#### Parameters

| Parameter | Type | Location | Description |
|-----------|------|----------|-------------|
| `dataSetId` | string | path | The data set ID to query |

#### Response

```json
{
  "cdnEgressQuota": "1099511627776",
  "cacheMissEgressQuota": "549755813888"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `cdnEgressQuota` | string | Remaining CDN egress quota in bytes |
| `cacheMissEgressQuota` | string | Remaining cache miss egress quota in bytes |

#### Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 404 | Data set not found |
| 405 | Method not allowed (use GET) |
| 500 | Internal server error |

#### Example

```bash
curl https://calibration.stats.filbeam.io/data-set/12345
```

```json
{
  "cdnEgressQuota": "1099511627776",
  "cacheMissEgressQuota": "549755813888"
}
```

---

### Get Payer Stats

Retrieve aggregated statistics across all data sets for a payer address.

```
GET /payer/{payerAddress}
```

#### Parameters

| Parameter | Type | Location | Description |
|-----------|------|----------|-------------|
| `payerAddress` | string | path | The payer's Ethereum address (0x...) |

**Note**: The address is normalized to lowercase.

#### Response

```json
{
  "totalRequests": "15234",
  "cacheMissRequests": "3847",
  "totalEgressBytes": "52428800000",
  "cacheMissEgressBytes": "13107200000",
  "remainingCDNEgressBytes": "1047483648000",
  "remainingCacheMissEgressBytes": "523741824000"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `totalRequests` | string | Total number of retrieval requests |
| `cacheMissRequests` | string | Number of requests that were cache misses |
| `totalEgressBytes` | string | Total bytes served |
| `cacheMissEgressBytes` | string | Bytes served from cache misses |
| `remainingCDNEgressBytes` | string | Remaining CDN quota across all data sets |
| `remainingCacheMissEgressBytes` | string | Remaining cache miss quota across all data sets |

#### Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 404 | Payer not found (no data sets) |
| 405 | Method not allowed (use GET) |
| 500 | Internal server error |

#### Example

```bash
curl https://calibration.stats.filbeam.io/payer/0x1234567890abcdef1234567890abcdef12345678
```

```json
{
  "totalRequests": "15234",
  "cacheMissRequests": "3847",
  "totalEgressBytes": "52428800000",
  "cacheMissEgressBytes": "13107200000",
  "remainingCDNEgressBytes": "1047483648000",
  "remainingCacheMissEgressBytes": "523741824000"
}
```
