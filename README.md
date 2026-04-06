# 🚀 ROBO PERF-ANALYZER v1.0

A powerful single-page web performance profiler built with .NET Core 8 MVC/Razor Pages.

## Features

✨ **Modern Web Performance Analysis**
- Single-page interface with clean, modern UI
- Real-time website performance profiling
- Headless browser automation using Microsoft Playwright
- Comprehensive resource timing analysis

📊 **Detailed Metrics**
- Total page load time measurement
- Total resource size calculation
- Per-category breakdown (Scripts, Styles, Images, Other)
- Individual asset analysis with response times
- Critical resource identification

🎯 **Critical Resource Detection**
- Flags JavaScript/CSS files > 300KB
- Identifies resources with load time > 500ms
- Visual warnings for performance issues

## Architecture

### Project Structure
```
PERF-ANALYZER - UPGRADE/
├── Controllers/
│   └── AnalyzerController.cs          # Main controller handling analysis
├── Models/
│   └── PerformanceReport.cs           # Data models and view models
├── Services/
│   └── PerformanceAnalysisService.cs  # Core analysis logic
├── Views/
│   ├── Analyzer/
│   │   └── Index.cshtml               # Main UI with Vanilla CSS
│   └── Shared/
│       ├── _Layout.cshtml
│       ├── Error.cshtml
│       └── _ViewStart.cshtml
├── Program.cs                          # ASP.NET configuration
├── appsettings.json                   # Configuration files
└── PerfAnalyzer.csproj                # Project file
```

### Technology Stack
- **Framework**: .NET Core 8 with ASP.NET MVC/Razor Pages
- **Browser Automation**: Microsoft Playwright (Chromium)
- **Frontend**: Vanilla CSS with responsive design
- **Architecture**: MVC pattern with Service layer

## How It Works

### 1. User Input
User enters a website URL in the input field and clicks "ANALYZE SITE"

### 2. Analysis Process
```csharp
PerformanceAnalysisService.AnalyzePerformanceAsync(url)
├── Initialize Playwright browser
├── Navigate to URL with WaitUntilState.NetworkIdle
├── Capture all Page.Response events
│   ├── Extract URL
│   ├── Determine ResourceType (JS/CSS/IMG/Other)
│   ├── Get Content-Length
│   └── Record ResponseTime
├── Categorize resources
├── Calculate metrics
├── Identify critical resources
└── Cleanup browser resources
```

### 3. Metrics Calculation

**Overall Metrics:**
- `TotalLoadTime`: Full page navigation completion time
- `TotalSize`: Sum of all resource sizes
- `TotalRequests`: Count of all HTTP requests

**Category Metrics:**
- Count of files per category
- Total size per category
- Cumulative load time per category
- Critical flag if conditions met

**Critical Condition:**
- JS/CSS file > 300KB → CRITICAL
- Load time > 500ms → CRITICAL

### 4. Output Display
Results displayed in:
- **Summary Cards**: Load time, total size, request count
- **Category Cards**: Per-category breakdown with status
- **Waterfall Table**: Detailed asset listing sorted by load time

## Installation & Setup

### Prerequisites
- .NET 8 SDK or later
- Windows/Linux/macOS with supported browser

### Steps

1. **Navigate to project directory**
```powershell
cd "d:\Research\PERF-ANALYZER - UPGRADE"
```

2. **Restore NuGet packages**
```bash
dotnet restore
```

3. **Install Playwright browsers** (one-time setup)
```bash
dotnet build PerfAnalyzer.csproj
# Playwright will auto-install on first run, or manually:
# pwsh bin/Debug/net8.0/playwright.ps1 install
```

4. **Run the application**
```bash
dotnet run --project PerfAnalyzer.csproj
```

5. **Open in browser**
```
http://localhost:5000
```

## Usage

### Basic Analysis
1. Enter a website URL (e.g., `https://example.com`)
2. Click "ANALYZE SITE" button
3. Wait for analysis to complete (typically 5-15 seconds)
4. View detailed performance metrics

### Interpreting Results

**Load Time Metric:**
- < 1s: Excellent
- 1-3s: Good
- 3-5s: Fair
- > 5s: Needs optimization

**Resource Size:**
- Files > 300KB marked as CRITICAL
- Indicates need for compression or splitting

**Critical Badge:**
- ⚠️ CRITICAL: Take action to optimize
- ✓ OK: Resource within acceptable parameters

## Performance Considerations

### Browser Lifecycle
The service properly manages browser resources:
```csharp
try
{
    // Initialize and use browser
}
finally
{
    // Ensure cleanup
    await page.DisposeAsync();
    await browser.CloseAsync();
    playwright.Dispose();
}
```

### Network Simulation
- Uses `WaitUntilState.NetworkIdle` for accurate measurements
- 30-second timeout per URL
- Ignores HTTPS errors for testing self-signed certs

### Concurrent Analysis
- Currently single-threaded (sequential)
- Can be extended with async queue system for production

## API Endpoints

### GET /Analyzer/Index
- Returns empty form and default view

### POST /Analyzer/Analyze
**Parameters:**
- `url` (required): Website URL to analyze

**Returns:**
- `PerformanceReport` model with analysis results

**Error Handling:**
- Validates URL format
- Returns error message on failure
- Preserves user input on error

## Customization

### Adjust Critical Thresholds
Edit `Services/PerformanceAnalysisService.cs`:
```csharp
bool isCritical = (asset.ResourceType == "Script" && asset.Size > 500_000) || // 500KB instead of 300KB
                (asset.ResourceType == "Stylesheet" && asset.Size > 500_000) ||
                asset.ResponseTime > 1000; // 1s instead of 500ms
```

### Modify Browser Options
Edit browser launch configuration:
```csharp
browser = await playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions
{
    Headless = false, // Show browser window
    SlowMo = 100,     // Slow down execution for debugging
});
```

### UI Customization
All styling in `Views/Analyzer/Index.cshtml` using CSS variables:
```css
:root {
    --primary: #6366f1;      /* Primary color */
    --danger: #ef4444;       /* Error/Critical color */
    --bg: #0f172a;           /* Background */
}
```

## Troubleshooting

### "Failed to install Playwright browsers"
```bash
# Manual installation
dotnet tool install -g Microsoft.Playwright.CLI
playwright install chromium
```

### Application won't start
```bash
# Clean and rebuild
dotnet clean PerfAnalyzer.csproj
dotnet build PerfAnalyzer.csproj
dotnet run --project PerfAnalyzer.csproj
```

### Analysis hangs on certain URLs
- Some sites may block automated access
- Try adding User-Agent header in Playwright context
- Increase timeout in `AnalyzePerformanceAsync` method

### Memory leaks with long-running analysis
- Ensure proper cleanup in finally block
- Monitor Process Memory while running tests
- Consider adding connection pooling for future versions

## Future Enhancements

- [ ] Concurrent analysis queue
- [ ] Performance history and trends
- [ ] Custom metrics rules
- [ ] Export reports (PDF/CSV)
- [ ] Core Web Vitals integration
- [ ] Lighthouse integration
- [ ] Network throttling simulation
- [ ] Database storage for reports

## License

This project is provided as-is for educational and research purposes.

## Support

For issues or questions about the analyzer:
1. Check browser console for client-side errors
2. Review application logs in `Properties/launchSettings.json`
3. Verify Playwright installation with `playwright --version`

---

**Built with ⚡ for performance analysis**
