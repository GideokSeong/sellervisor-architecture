# Background Job Example

This example shows how SellerVisor handles long-running Amazon Ads sync work outside the request/response cycle.

## Why Background Jobs Are Used

Amazon Ads data sync can take time because it involves:

- External API calls
- Date-range batching
- Rate limits
- Database inserts/updates
- Retry handling

Instead of making the user wait inside an HTTP request, the controller only enqueues the job.

---

## Controller

```csharp
[HttpPost]
public IActionResult SyncAdsMetrics(DateTime startDate, DateTime endDate)
{
    var tenantId = _tenantProvider.TenantId;

    _backgroundJobClient.Enqueue<IAdsSyncJob>(job =>
        job.SyncAdsMetricsAsync(tenantId, startDate, endDate));

    return Ok(new
    {
        success = true,
        message = "Ads sync job has been queued."
    });
}
```

---

## Job Interface

```csharp
public interface IAdsSyncJob
{
    Task SyncAdsMetricsAsync(
        int tenantId,
        DateTime startDate,
        DateTime endDate);
}
```

---

## Job Implementation

```csharp
public class AdsSyncJob : IAdsSyncJob
{
    private readonly IAmazonAdsApiService _amazonAdsApiService;
    private readonly ApplicationDbContext _context;
    private readonly ILogger<AdsSyncJob> _logger;

    public AdsSyncJob(
        IAmazonAdsApiService amazonAdsApiService,
        ApplicationDbContext context,
        ILogger<AdsSyncJob> logger)
    {
        _amazonAdsApiService = amazonAdsApiService;
        _context = context;
        _logger = logger;
    }

    public async Task SyncAdsMetricsAsync(
        int tenantId,
        DateTime startDate,
        DateTime endDate)
    {
        try
        {
            var currentDate = startDate.Date;

            while (currentDate <= endDate.Date)
            {
                var metrics = await _amazonAdsApiService
                    .FetchCampaignMetricsAsync(tenantId, currentDate);

                foreach (var metric in metrics)
                {
                    var existing = await _context.AmazonAdsMetrics
                        .FirstOrDefaultAsync(x =>
                            x.TenantId == tenantId &&
                            x.CampaignId == metric.CampaignId &&
                            x.Date == currentDate);

                    if (existing == null)
                    {
                        _context.AmazonAdsMetrics.Add(new AmazonAdsMetric
                        {
                            TenantId = tenantId,
                            CampaignId = metric.CampaignId,
                            CampaignName = metric.CampaignName,
                            Date = currentDate,
                            Impressions = metric.Impressions,
                            Clicks = metric.Clicks,
                            Cost = metric.Cost,
                            Sales1d = metric.Sales1d,
                            Sales7d = metric.Sales7d,
                            Purchases1d = metric.Purchases1d,
                            Purchases7d = metric.Purchases7d
                        });
                    }
                    else
                    {
                        existing.Impressions = metric.Impressions;
                        existing.Clicks = metric.Clicks;
                        existing.Cost = metric.Cost;
                        existing.Sales1d = metric.Sales1d;
                        existing.Sales7d = metric.Sales7d;
                        existing.Purchases1d = metric.Purchases1d;
                        existing.Purchases7d = metric.Purchases7d;
                    }
                }

                await _context.SaveChangesAsync();

                currentDate = currentDate.AddDays(1);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex,
                "Failed to sync ads metrics. TenantId: {TenantId}, StartDate: {StartDate}, EndDate: {EndDate}",
                tenantId,
                startDate,
                endDate);

            throw;
        }
    }
}
```

---

## Why This Pattern Works

- HTTP request stays fast
- Long-running work is isolated
- Failures are logged in one place
- Jobs can be retried safely
- External API calls do not block the UI
