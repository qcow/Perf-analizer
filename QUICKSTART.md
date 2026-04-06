# 🚀 Quick Start Guide - ROBO PERF-ANALYZER

## Installation & Running (30 seconds)

### 1. Open PowerShell / Terminal
```powershell
cd "d:\Research\PERF-ANALYZER - UPGRADE"
```

### 2. Install Playwright Browsers (First time only)
```powershell
dotnet build PerfAnalyzer.csproj
```

### 3. Run the Application
```powershell
dotnet run --project PerfAnalyzer.csproj
```

The app will start at **http://localhost:5000**

### 4. Open in Browser
Navigate to `http://localhost:5000` and you'll see the analyzer interface.

---

## How to Use

### Step 1: Enter URL
```
https://example.com
https://github.com
https://wikipedia.org
```

### Step 2: Click "ANALYZE SITE"
The browser will:
- Navigate to your URL
- Wait for network idle (all resources loaded)
- Capture all HTTP responses
- Measure load times

### Step 3: Review Results
You'll see:
- **Load Time**: Total milliseconds to fully load
- **Total Size**: Sum of all assets (compressed)
- **Requests**: Number of HTTP requests

### Step 4: Identify Issues
Look for:
- **⚠️ CRITICAL** badges = Performance issues
- Files > 300KB = Need optimization
- Load times > 500ms = Slow resources

---

## Example Analysis

### Input
```
https://example.com
```

### Output
```
LOAD TIME:     1,245ms
TOTAL SIZE:    2.5 MB
REQUESTS:      42

SCRIPTS:       ⚠️ CRITICAL
  - Files: 3
  - Size: 450KB (large!)
  - Duration: 650ms

STYLES:        ✓ OK
  - Files: 1
  - Size: 85KB
  - Duration: 120ms

IMAGES:        ✓ OK
  - Files: 15
  - Size: 1.8MB
  - Duration: 450ms

WATERFALL TABLE (Sorted by Load Time)
[Detailed list of each asset]
```

---

## Architecture Overview

```
REQUEST
   |
   V
AnalyzerController.Analyze()
   |
   V
PerformanceAnalysisService.AnalyzePerformanceAsync()
   |
   +-- Launch Playwright Chromium Browser
   +-- Navigate to URL with WaitUntilState.NetworkIdle
   +-- Capture page.Response events
   |   ├-- Extract URL
   |   ├-- Determine ResourceType
   |   ├-- Get Content-Length from headers
   |   └-- Record ResponseTime
   +-- Process & Categorize Resources
   +-- Calculate Metrics
   +-- Clean up Browser
   |
   V
Return PerformanceReport Model
   |
   V
Index.cshtml Renders Results
   |
   V
User Sees Beautiful Dashboard
```

---

## Critical Conditions

An asset is marked **⚠️ CRITICAL** if:

1. **JavaScript File > 300KB**
   - Impacts Time-to-Interactive
   - Consider code splitting

2. **CSS File > 300KB**
   - Blocks rendering
   - Consider modular CSS

3. **Any Resource > 500ms Load Time**
   - Network issues or large file
   - Check CDN configuration

---

## Troubleshooting

### App won't start
```powershell
# Clean build
dotnet clean PerfAnalyzer.csproj
dotnet build PerfAnalyzer.csproj
dotnet run --project PerfAnalyzer.csproj
```

### Port already in use
Change port in `Properties/launchSettings.json`:
```json
"applicationUrl": "http://localhost:5050"
```

### Analysis hangs on certain sites
- Some sites block automation
- Try different domain
- Check firewall/proxy

### Playwright not installed
```powershell
pwsh bin/Debug/net8.0/playwright.ps1 install chromium
```

---

## File Structure

```
PERF-ANALYZER - UPGRADE/
├── Controllers/
│   └── AnalyzerController.cs       (HTTP endpoints)
├── Models/
│   └── PerformanceReport.cs        (Data models)
├── Services/
│   └── PerformanceAnalysisService.cs (Core logic)
├── Views/
│   ├── Analyzer/
│   │   └── Index.cshtml            (Main UI - Vanilla CSS)
│   └── Shared/
│       ├── _Layout.cshtml
│       ├── _ViewStart.cshtml
│       └── Error.cshtml
├── Program.cs                       (Startup config)
├── appsettings.json                (Settings)
└── PerfAnalyzer.csproj             (Project file)
```

---

## Performance Tips

### For Testing Large Sites
- Allow 10-15 seconds per analysis
- Test during off-peak hours
- Monitor local resource usage

### For Batch Analysis
Currently sequential - enhance with:
- Queue system
- Concurrent browser instances
- Database storage

---

## Support

**Getting help:**

1. Check application output in terminal
2. Look for ⚠️ or ❌ error messages
3. Check browser developer console (F12)
4. Verify URL is accessible
5. Ensure internet connection

---

## Next Steps

Try analyzing these popular sites:
- https://www.google.com
- https://www.github.com
- https://www.wikipedia.org
- https://www.stackoverflow.com

Compare their performance metrics!

---

**⚡ Happy Profiling!**
