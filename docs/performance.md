# Performance

## Problem

Heavy aggregation queries caused:

- High CPU usage
- Slow response times

---

## Example Query Pattern

Filtering by:
- TenantId
- Date range

Aggregating:
- Sales
- Clicks
- Impressions

---

## Solutions

### 1. Indexing

Composite indexes:

- (TenantId, Date)
- (CampaignId, Date)

---

### 2. Query Optimization

- Reduced projections
- Avoided unnecessary joins
- Used selective columns

---

### 3. Async Processing

Heavy operations moved to background jobs

---

## Result

- Faster reporting
- Stable performance under load
