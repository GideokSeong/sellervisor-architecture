# Data Flow

## Overview

The system processes Amazon Ads data in batches.

---

## Flow

1. Fetch data from Amazon API
2. Normalize into database tables
3. Store raw metrics
4. Aggregate on query

---

## Key Tables

- AmazonAdsMetric
- AmazonKeywordMetric

Each record includes:
- Date
- Campaign / Keyword identifiers
- Performance metrics

---

## Design Decision

Aggregation is done at query time instead of write time.

### Why?

- Flexibility
- Avoid duplicated data

### Tradeoff

- Heavier queries
→ solved with indexing
