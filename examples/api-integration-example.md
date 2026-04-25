# External API Integration Example

This example shows how external API logic is isolated inside a service instead of being placed directly in controllers.

## Why This Matters

External APIs can be unreliable:

- Rate limits
- Token expiration
- Timeout issues
- Different response formats
- Partial failures

Keeping API logic inside a service makes it easier to test, retry, and maintain.

---

## Service Interface

```csharp
public interface IAmazonAdsApiService
{
    Task<IReadOnlyList<AmazonAdsMetricDto>> FetchCampaignMetricsAsync(
        int tenantId,
        DateTime date);
}
```

---

## DTO

```csharp
public class AmazonAdsMetricDto
{
    public string CampaignId { get; set; } = string.Empty;
    public string CampaignName { get; set; } = string.Empty;

    public long Impressions { get; set; }
    public long Clicks { get; set; }

    public decimal Cost { get; set; }
    public decimal Sales1d { get; set; }
    public decimal Sales7d { get; set; }

    public int Purchases1d { get; set; }
    public int Purchases7d { get; set; }
}
```

---

## API Service

```csharp
public class AmazonAdsApiService : IAmazonAdsApiService
{
    private readonly HttpClient _httpClient;
    private readonly IAmazonTokenService _tokenService;
    private readonly ILogger<AmazonAdsApiService> _logger;

    public AmazonAdsApiService(
        HttpClient httpClient,
        IAmazonTokenService tokenService,
        ILogger<AmazonAdsApiService> logger)
    {
        _httpClient = httpClient;
        _tokenService = tokenService;
        _logger = logger;
    }

    public async Task<IReadOnlyList<AmazonAdsMetricDto>> FetchCampaignMetricsAsync(
        int tenantId,
        DateTime date)
    {
        var accessToken = await _tokenService.GetAccessTokenAsync(tenantId);

        using var request = new HttpRequestMessage(
            HttpMethod.Post,
            "/reporting/campaigns");

        request.Headers.Authorization =
            new AuthenticationHeaderValue("Bearer", accessToken);

        var payload = new
        {
            startDate = date.ToString("yyyy-MM-dd"),
            endDate = date.ToString("yyyy-MM-dd"),
            metrics = new[]
            {
                "campaignId",
                "campaignName",
                "impressions",
                "clicks",
                "cost",
                "sales1d",
                "sales7d",
                "purchases1d",
                "purchases7d"
            }
        };

        request.Content = JsonContent.Create(payload);

        using var response = await _httpClient.SendAsync(request);

        if (!response.IsSuccessStatusCode)
        {
            var errorBody = await response.Content.ReadAsStringAsync();

            _logger.LogWarning(
                "Amazon Ads API failed. TenantId: {TenantId}, StatusCode: {StatusCode}, Body: {Body}",
                tenantId,
                response.StatusCode,
                errorBody);

            throw new ExternalApiException(
                "Amazon Ads API request failed.");
        }

        var result = await response.Content
            .ReadFromJsonAsync<List<AmazonAdsMetricDto>>();

        return result ?? new List<AmazonAdsMetricDto>();
    }
}
```

---

## Controller Usage

```csharp
[HttpGet]
public async Task<IActionResult> CampaignMetrics(DateTime date)
{
    var tenantId = _tenantProvider.TenantId;

    var metrics = await _amazonAdsApiService
        .FetchCampaignMetricsAsync(tenantId, date);

    return Ok(metrics);
}
```

---

## Key Design Decisions

- Controllers do not know API details
- Token handling is separated
- DTOs are explicit
- Failures are logged with context
- External API errors are not swallowed silently
