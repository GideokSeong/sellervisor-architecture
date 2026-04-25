# Query Optimization Example

This example shows a common performance issue in reporting pages and how it was improved.

## Problem

The reporting dashboard needs to aggregate Amazon Ads data by ASIN.

The original query worked for small datasets, but became expensive as data volume increased.

---

## Original Query

```csharp
var metrics = await _context.AmazonAdsMetrics
    .Where(x =>
        x.TenantId == tenantId &&
        x.Date >= startDate &&
        x.Date <= endDate)
    .ToListAsync();

var result = metrics
    .GroupBy(x => x.AdvertisedAsin)
    .Select(g => new AdsSummaryDto
    {
        AdvertisedAsin = g.Key,
        Impressions = g.Sum(x => x.Impressions),
        Clicks = g.Sum(x => x.Clicks),
        Cost = g.Sum(x => x.Cost),
        Sales = g.Sum(x => x.Sales7d > 0 ? x.Sales7d : x.Sales1d),
        Orders = g.Sum(x => x.Purchases7d > 0 ? x.Purchases7d : x.Purchases1d)
    })
    .ToList();
```

---

## Issue

This query pulls all matching rows into memory before grouping.

Problems:

- High memory usage
- More data transferred from SQL Server
- Slower response time
- Higher CPU usage on the app server

---

## Optimized Query

```csharp
var result = await _context.AmazonAdsMetrics
    .AsNoTracking()
    .Where(x =>
        x.TenantId == tenantId &&
        x.Date >= startDate &&
        x.Date <= endDate)
    .GroupBy(x => x.AdvertisedAsin)
    .Select(g => new AdsSummaryDto
    {
        AdvertisedAsin = g.Key,

        Impressions = g.Sum(x => x.Impressions),
        Clicks = g.Sum(x => x.Clicks),
        Cost = g.Sum(x => x.Cost),

        Sales = g.Sum(x =>
            x.Sales7d > 0
                ? x.Sales7d
                : x.Sales1d),

        Orders = g.Sum(x =>
            x.Purchases7d > 0
                ? x.Purchases7d
                : x.Purchases1d)
    })
    .OrderByDescending(x => x.Cost)
    .ToListAsync();
```

---

## DTO

```csharp
public class AdsSummaryDto
{
    public string? AdvertisedAsin { get; set; }

    public long Impressions { get; set; }
    public long Clicks { get; set; }

    public decimal Cost { get; set; }
    public decimal Sales { get; set; }

    public int Orders { get; set; }

    public decimal Acos =>
        Sales > 0
            ? Cost / Sales * 100
            : 0;

    public decimal Ctr =>
        Impressions > 0
            ? (decimal)Clicks / Impressions * 100
            : 0;

    public decimal ConversionRate =>
        Clicks > 0
            ? (decimal)Orders / Clicks * 100
            : 0;
}
```

---

## Index Used

```csharp
modelBuilder.Entity<AmazonAdsMetric>()
    .HasIndex(x => new
    {
        x.TenantId,
        x.Date,
        x.AdvertisedAsin
    });
```

---

## Why This Is Better

- SQL Server performs the grouping
- App server receives only aggregated results
- EF tracking is disabled with `AsNoTracking`
- Query uses indexed filter columns
- Less memory pressure

---

## Notes

This does not mean every query should be pushed fully into SQL.

For reporting workloads, database-side aggregation is usually better because the database engine is optimized for filtering, grouping, and aggregation.
