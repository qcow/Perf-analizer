# Technical Documentation - ROBO PERF-ANALYZER

## System Architecture

### Application Layers

```
┌─────────────────────────────────────┐
│     Views (Razor Pages + CSS)       │
│  - Index.cshtml (main interface)    │
│  - Modern vanilla CSS styling       │
│  - Real-time feedback               │
└──────────────┬──────────────────────┘
               │ (HTTP POST)
               V
┌─────────────────────────────────────┐
│       Controllers (MVC)              │
│  - AnalyzerController               │
│  - HTTP routing                     │
│  - URL validation                   │
└──────────────┬──────────────────────┘
               │ (Models)
               V
┌─────────────────────────────────────┐
│       Services (Business Logic)      │
│  - PerformanceAnalysisService       │
│  - Browser automation               │
│  - Metrics calculation              │
│  - Resource categorization          │
└──────────────┬──────────────────────┘
               │
               V
┌─────────────────────────────────────┐
│    Playwright (Browser Automation)   │
│  - Chromium browser instance        │
│  - Page navigation                  │
│  - Response interception            │
│  - Resource timing capture          │
└─────────────────────────────────────┘
```

---

## Data Flow

### Analysis Process Flow

```
User Input (URL)
    |
    V
[AnalyzerController.Analyze()]
    |
    +-- URL Validation
    |   ├-- Format check (http/https)
    |   ├-- Add https:// if needed
    |   └-- Uri.TryCreate() validation
    |
    V
[PerformanceAnalysisService.AnalyzePerformanceAsync(url)]
    |
    +-- Browser Initialization
    |   ├-- Playwright.CreateAsync()
    |   ├-- LaunchAsync(Chromium)
    |   ├-- NewContextAsync()
    |   └-- NewPageAsync()
    |
    +-- Event Listeners
    |   └-- page.Response += (sender, response) { ... }
    |
    +-- Page Navigation
    |   └-- page.GotoAsync(url, WaitUntilState.NetworkIdle)
    |
    +-- Response Capture (per asset)
    |   ├-- Extract URL
    |   ├-- Get headers (Content-Length)
    |   ├-- Determine ResourceType
    |   ├-- Record ResponseTime
    |   └-- Store AssetMetric
    |
    +-- Metrics Calculation
    |   ├-- Total metrics (size, time, count)
    |   ├-- Category breakdown
    |   ├-- Critical detection
    |   └-- Asset sorting
    |
    +-- Cleanup
    |   ├-- page.CloseAsync()
    |   ├-- context.CloseAsync()
    |   └-- browser.CloseAsync()
    |
    V
Return PerformanceReport Model
    |
    V
[Index.cshtml View]
    |
    +-- Render Summary Cards
    |   ├-- Load Time
    |   ├-- Total Size
    |   └-- Request Count
    |
    +-- Render Category Cards
    |   ├-- Scripts (with critical flag)
    |   ├-- Styles (with critical flag)
    |   ├-- Images (with critical flag)
    |   └-- Other (with critical flag)
    |
    +-- Render Waterfall Table
    |   └-- Asset list (sorted by load time)
    |
    V
Display to User
```

---

## Key Components

### 1. AnalyzerController.cs

```csharp
public class AnalyzerController : Controller
{
    private readonly PerformanceAnalysisService _analysisService;

    // GET /Analyzer/Index
    // Returns empty form
    public IActionResult Index();

    // POST /Analyzer/Analyze
    // Performs analysis and returns results
    public async Task<IActionResult> Analyze(string url);
}
```

**Responsibilities:**
- Route HTTP requests
- Validate input URLs
- Call analysis service
- Handle errors gracefully
- Return view with model

---

### 2. PerformanceAnalysisService.cs

```csharp
public class PerformanceAnalysisService
{
    // Main analysis entry point
    public async Task<PerformanceReport> AnalyzePerformanceAsync(string url);

    // Helper methods
    private void PopulateReport(PerformanceReport report, 
                                List<AssetMetric> assets, 
                                double totalTime);
    
    private string DetermineResourceType(string url);
    private string ExtractFileName(string url);
}
```

**Responsibilities:**
- Browser lifecycle management
- Response event handling
- Metrics calculation
- Resource categorization
- Critical detection
- Data cleanup

---

### 3. PerformanceReport.cs (Models)

```csharp
public class PerformanceReport
{
    // Overall metrics
    public double TotalLoadTime { get; set; }      // ms
    public long TotalSize { get; set; }            // bytes
    public int TotalRequests { get; set; }         // count
    
    // Category breakdowns
    public CategoryMetrics Scripts { get; set; }
    public CategoryMetrics Styles { get; set; }
    public CategoryMetrics Images { get; set; }
    public CategoryMetrics Other { get; set; }
    
    // Detailed assets
    public List<AssetMetric> Assets { get; set; }
}

public class CategoryMetrics
{
    public long TotalSize { get; set; }
    public double TotalDuration { get; set; }
    public int Count { get; set; }
    public bool IsCritical { get; set; }
}

public class AssetMetric
{
    public string FileName { get; set; }           // e.g., jquery.min.js
    public string ResourceType { get; set; }       // Script, Stylesheet, Image, Other
    public long Size { get; set; }                 // bytes
    public double ResponseTime { get; set; }       // ms
    public int StatusCode { get; set; }            // HTTP status
    public bool IsCritical { get; set; }           // Performance flag
}
```

---

### 4. Index.cshtml (View)

**Structure:**
- Header with input form
- Error alert (if needed)
- Summary metrics (3 cards)
- Category breakdown (4 cards)
- Waterfall table (detailed assets)

**Styling:**
- Vanilla CSS (no frameworks)
- CSS variables for theming
- Responsive grid layout
- Dark theme with accent colors
- Smooth transitions and animations

---

## Dependencies

### NuGet Packages

```xml
<ItemGroup>
    <PackageReference Include="Microsoft.Playwright" Version="1.40.0" />
</ItemGroup>
```

### Framework
- .NET Core 8.0
- ASP.NET MVC
- Razor Pages

### Runtime Requirements
- Chromium browser (auto-installed)
- ~200MB disk space

---

## Performance Characteristics

### Time Complexity
- URL validation: O(1)
- Page load: O(n) where n = number of resources
- Resource categorization: O(n)
- Sorting: O(n log n)
- **Total: O(n log n)**

### Space Complexity
- AssetMetric per resource: ~200 bytes
- Total for 42 resources: ~8.4 KB
- Report overhead: ~500 bytes
- **Total: O(n)** where n = resource count

### Actual Performance

| Operation | Time |
|-----------|------|
| URL validation | < 1ms |
| Browser startup | 1-2 sec |
| Page navigation | 5-15 sec (varies) |
| Response capture | < 10ms per asset |
| Data processing | < 100ms |
| View rendering | < 50ms |
| **Total** | **5-20 seconds** |

---

## Error Handling Strategy

### Error Types

```
1. Input Validation Errors
   - Empty URL
   - Invalid format
   → User-friendly message

2. Network Errors
   - Site unreachable
   - Timeout
   → "Analysis failed: [reason]"

3. Browser Errors
   - Playwright not installed
   - Browser crash
   → Exception with details

4. Processing Errors
   - Invalid response
   - Metric calculation failure
   → "Error during analysis"
```

### Error Recovery

```csharp
try
{
    // Analysis logic
}
catch (Exception ex)
{
    var errorReport = new PerformanceReport 
    { 
        ErrorMessage = $"Error during analysis: {ex.Message}"
    };
    return View("Index", errorReport);
}
finally
{
    // Cleanup ALWAYS happens
    // Browser/context/page disposal guaranteed
}
```

---

## Browser Lifecycle Management

### Initialization

```csharp
IPlaywright playwright = await Playwright.CreateAsync();
IBrowser browser = await playwright.Chromium.LaunchAsync(
    new BrowserTypeLaunchOptions { Headless = true }
);
var context = await browser.NewContextAsync(
    new BrowserNewContextOptions { IgnoreHTTPSErrors = true }
);
IPage page = await context.NewPageAsync();
```

### Event Listening

```csharp
page.Response += (sender, response) =>
{
    // Called for each HTTP response
    var url = response.Url;
    var headers = response.Headers;
    var status = response.Status;
    var contentLength = long.Parse(headers["content-length"]);
    // Record metric
};
```

### Navigation

```csharp
await page.GotoAsync(url, new PageGotoOptions 
{ 
    WaitUntilState = WaitUntilState.NetworkIdle,
    Timeout = 30000  // 30 seconds
});
```

### Cleanup

```csharp
finally
{
    if (page != null) await page.CloseAsync();
    if (browser != null) await browser.CloseAsync();
    if (playwright != null) playwright.Dispose();
}
```

**Key Point:** All resources cleaned up via finally block, even if errors occur.

---

## Resource Type Detection Algorithm

```csharp
private string DetermineResourceType(string url)
{
    // Parse URL to get path
    var uri = new Uri(url);
    var path = uri.AbsolutePath.ToLower();
    var extension = Path.GetExtension(path);
    
    // Match extension
    return extension switch
    {
        ".js" => "Script",
        ".css" => "Stylesheet",
        ".jpg" or ".jpeg" or ".png" or ".gif" or ".webp" or ".svg" or ".ico" => "Image",
        _ => "Other"
    };
}
```

**Coverage:**
- Scripts: .js
- Stylesheets: .css
- Images: .jpg, .jpeg, .png, .gif, .webp, .svg, .ico
- Other: All remaining types

---

## Critical Detection Logic

```csharp
bool isCritical = 
    // Large JavaScript
    (asset.ResourceType == "Script" && asset.Size > 300_000) ||
    
    // Large CSS
    (asset.ResourceType == "Stylesheet" && asset.Size > 300_000) ||
    
    // Slow resource (any type)
    asset.ResponseTime > 500;
```

**Thresholds:**
- JS/CSS size: 300 KB (300,000 bytes)
- Load time: 500 ms

---

## Configuration

### appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### launchSettings.json

```json
{
  "profiles": {
    "http": {
      "applicationUrl": "http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

---

## Security Considerations

### Current (Development)
- No authentication
- No HTTPS enforcement (except in HTTPS profile)
- No rate limiting
- No request logging

### Production Recommendations
- [ ] Add authentication/authorization
- [ ] Implement rate limiting (queue system)
- [ ] Add request validation
- [ ] Implement audit logging
- [ ] Sanitize URLs
- [ ] Add CORS configuration
- [ ] Use HTTPS only
- [ ] Add API key validation
- [ ] Implement timeout per user
- [ ] Add IP whitelisting

---

## Testing Strategy

### Unit Tests (Needed)
```csharp
[TestClass]
public class PerformanceAnalysisServiceTests
{
    [TestMethod]
    public void DetermineResourceType_JsFile_ReturnsScript()
    {
        // Arrange
        var service = new PerformanceAnalysisService();
        
        // Act
        var result = service.DetermineResourceType(
            "https://example.com/jquery.min.js"
        );
        
        // Assert
        Assert.AreEqual("Script", result);
    }
}
```

### Integration Tests (Needed)
```csharp
[TestClass]
public class AnalyzerControllerTests
{
    [TestMethod]
    public async Task Analyze_ValidUrl_ReturnsResults()
    {
        // Test actual analysis
    }
}
```

### Manual Testing
1. Navigate to http://localhost:5000
2. Enter URL: https://example.com
3. Click "ANALYZE SITE"
4. Verify results display
5. Check for CRITICAL badges

---

## Deployment

### Local Development
```bash
dotnet run --project PerfAnalyzer.csproj
```

### Docker Deployment
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 as build
WORKDIR /app
COPY . .
RUN dotnet build PerfAnalyzer.csproj

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/bin/Release/net8.0 .
EXPOSE 5000
ENTRYPOINT ["dotnet", "PerfAnalyzer.dll"]
```

### IIS Deployment
```powershell
dotnet publish PerfAnalyzer.csproj -c Release -o .\publish
# Copy publish folder to IIS website directory
```

---

## Monitoring & Logging

### Current Status
- Basic ASP.NET logging
- Console output only
- No persistent logs

### Recommended Additions
- [ ] Structured logging (Serilog)
- [ ] Performance timing logs
- [ ] Error tracking (Application Insights)
- [ ] Request/response logging
- [ ] Analytics

---

## Future Enhancements

### High Priority
- [ ] Concurrent analysis queue
- [ ] Results caching
- [ ] Batch analysis endpoint
- [ ] Database storage
- [ ] Performance trends

### Medium Priority
- [ ] Export to PDF/CSV
- [ ] Core Web Vitals integration
- [ ] Lighthouse integration
- [ ] Network throttling
- [ ] Custom metrics

### Low Priority
- [ ] Mobile simulation
- [ ] Geolocation-based tests
- [ ] Scheduled reports
- [ ] Slack integration
- [ ] Custom branding

---

## Version History

### v1.0 (Current)
- Single-page analyzer
- URL input validation
- Browser automation via Playwright
- Performance metrics capture
- Modern dark UI
- Resource categorization
- Critical detection
- Waterfall view

---

**Last Updated:** March 24, 2026  
**Maintainer:** ROBO PERF-ANALYZER Team
