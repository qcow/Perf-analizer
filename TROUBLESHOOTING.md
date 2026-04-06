# Troubleshooting & Testing Guide

## Common Issues & Solutions

---

## Issue 1: "Playwright not installed" Error

### Symptoms
```
Exception: Failed to install Playwright browsers
```

### Solutions

**Option 1: Auto-install (Recommended)**
```powershell
cd "d:\Research\PERF-ANALYZER - UPGRADE"
dotnet build PerfAnalyzer.csproj
```

**Option 2: Manual Installation**
```powershell
# Install Playwright CLI tool
dotnet tool install -g Microsoft.Playwright.CLI

# Install Chromium browser
playwright install chromium
```

**Option 3: Clean Reinstall**
```powershell
dotnet clean PerfAnalyzer.csproj
dotnet restore
dotnet build PerfAnalyzer.csproj
```

---

## Issue 2: Port Already in Use

### Symptoms
```
System.Net.Sockets.SocketException: Only one usage of each socket address 
(protocol/port combination) is normally permitted
```

### Solutions

**Option 1: Kill Process on Port 5000**
```powershell
# Find process
Get-NetTCPConnection -LocalPort 5000 | Select-Object OwningProcess

# Kill process (replace PID with actual number)
Stop-Process -Id <PID> -Force
```

**Option 2: Use Different Port**
Edit `Properties/launchSettings.json`:
```json
"applicationUrl": "http://localhost:5050"
```

**Option 3: Wait 60 seconds**
The port may be in TIME_WAIT state. Just wait.

---

## Issue 3: Analysis Hangs

### Symptoms
- Browser opens but never closes
- Page stays "Analyzing..." for > 30 seconds
- Terminal shows no output

### Solutions

**Option 1: Increase Timeout**
Edit `Services/PerformanceAnalysisService.cs`:
```csharp
await page.GotoAsync(url, new PageGotoOptions 
{ 
    WaitUntilState = WaitUntilState.NetworkIdle,
    Timeout = 60000  // Increase to 60 seconds
});
```

**Option 2: Kill Browser Process**
```powershell
# In separate PowerShell
Get-Process | Where-Object {$_.ProcessName -eq "chrome"} | Stop-Process -Force
```

**Option 3: Try Different Website**
Some sites might block automation:
- Try: https://example.com
- Try: https://wikipedia.org
- Avoid: Sites that require login

---

## Issue 4: Invalid URL Format Error

### Symptoms
```
Error: Invalid URL format
```

### Valid Formats
```
✓ https://example.com
✓ http://example.com
✓ https://example.com:8080
✓ https://sub.example.com
✗ example.com (missing protocol)
✗ ftp://example.com (wrong protocol)
✗ htp://example.com (typo)
```

### Solution
Always include `http://` or `https://`
```
Good:  https://example.com
Bad:   example.com

App will auto-convert:
Input:  example.com
Output: https://example.com
```

---

## Issue 5: Analysis Fails with Timeout

### Symptoms
```
Error during analysis: timeout exceeded
```

### Causes
- Website is very slow
- Network issues
- Website blocks automation
- Server down

### Solutions

**Option 1: Increase Timeout** (see Issue 3)

**Option 2: Check Network**
```powershell
# Test connectivity
ping example.com
```

**Option 3: Test in Browser Manually**
- Open site in regular Chrome
- Time how long it loads
- If > 30 seconds, timeout is expected

---

## Issue 6: No Results in Waterfall Table

### Symptoms
- Metrics show but table is empty
- TotalRequests = 0
- No assets displayed

### Causes
- Site blocked response capture
- Page didn't navigate properly
- Network error

### Debug Steps
1. Check the URL in browser directly
2. Try simpler site (example.com)
3. Check browser DevTools in manual navigation

---

## Issue 7: Application Won't Start

### Symptoms
```
error CS0103: The name 'XYZ' does not exist
```

### Solutions

**Full Clean Rebuild**
```powershell
cd "d:\Research\PERF-ANALYZER - UPGRADE"

# Clean
dotnet clean PerfAnalyzer.csproj

# Restore
dotnet restore

# Build
dotnet build PerfAnalyzer.csproj

# Run
dotnet run --project PerfAnalyzer.csproj
```

**Check .NET Version**
```powershell
dotnet --version
# Should be 8.0.x or higher
```

**Update .NET**
```powershell
# If needed, download from dotnet.microsoft.com
```

---

## Issue 8: High Memory Usage

### Symptoms
- Memory keeps growing
- Process takes > 1GB RAM
- System becomes slow

### Causes
- Browser not closing properly
- Multiple browser instances
- Memory leak in analysis loop

### Solutions

**Option 1: Monitor Process**
```powershell
# In new PowerShell window
Get-Process PerfAnalyzer | 
  ForEach-Object { 
    Write-Host "Memory: $($_.WorkingSet / 1MB) MB"
    Start-Sleep -Seconds 2
  }
```

**Option 2: Restart Application**
```powershell
# Ctrl+C to stop
# Then restart
dotnet run --project PerfAnalyzer.csproj
```

**Option 3: Check Code**
Ensure finally block always runs:
```csharp
finally
{
    if (page != null) await page.CloseAsync();
    if (browser != null) await browser.CloseAsync();
}
```

---

## Testing Guide

### Manual Testing Checklist

```
□ Application Startup
  □ dotnet run --project PerfAnalyzer.csproj works
  □ No build errors
  □ Port 5000 accessible
  □ Page loads at http://localhost:5000

□ UI Elements
  □ Input field visible and focusable
  □ Analyze button visible and clickable
  □ Emoji icons display correctly
  □ Dark theme loads properly

□ Input Validation
  □ Empty URL shows error
  □ Invalid format shows error
  □ Valid URL accepted
  □ Auto-adds https:// if needed

□ Analysis Execution
  □ Button shows "Analyzing..." state
  □ Analysis completes in < 30 seconds
  □ No browser window appears
  □ No console errors

□ Results Display
  □ Summary metrics visible
  □ Load time shows in ms
  □ Total size formatted correctly (KB/MB)
  □ Request count accurate

□ Category Breakdown
  □ Scripts card visible (if JS files exist)
  □ Styles card visible (if CSS files exist)
  □ Images card visible (if images exist)
  □ CRITICAL badges appear correctly

□ Waterfall Table
  □ Assets listed with correct names
  □ File types displayed (Script, Stylesheet, Image)
  □ Sizes formatted as KB/MB
  □ Times shown in ms
  □ Status labels correct (✓ OK, ⚠️ CRITICAL)

□ Error Handling
  □ Bad URL shows error message
  □ Timeout handled gracefully
  □ Browser cleanup on error
  □ User can retry analysis
```

### Test URLs

**Good Test Sites**
```
https://example.com         (Simple, fast)
https://wikipedia.org       (Medium, many resources)
https://github.com          (Complex, many JS/CSS)
https://wikipedia.org/wiki/Main_Page  (Large site)
```

**Problematic Sites**
```
https://localhost:3000      (May not exist)
https://intranet.company    (May be blocked)
https://private-server      (Not public)
```

---

## Browser DevTools Debugging

### Enable Browser UI (For Debugging)

Edit `PerformanceAnalysisService.cs`:
```csharp
browser = await playwright.Chromium.LaunchAsync(
    new BrowserTypeLaunchOptions
    {
        Headless = false  // Show browser window
    }
);
```

Now you'll see the browser window during analysis.

### Slow Down Execution
```csharp
browser = await playwright.Chromium.LaunchAsync(
    new BrowserTypeLaunchOptions
    {
        SlowMo = 1000  // Wait 1 second between actions
    }
);
```

---

## Performance Testing

### Measure Analysis Time

Add timing code:
```csharp
var stopwatch = System.Diagnostics.Stopwatch.StartNew();

// Analysis code here

stopwatch.Stop();
Console.WriteLine($"Analysis took {stopwatch.ElapsedMilliseconds}ms");
```

### Test Various Sites

```powershell
$sites = @(
    "https://example.com",
    "https://wikipedia.org",
    "https://github.com"
)

foreach ($site in $sites) {
    Write-Host "Testing: $site"
    # Submit form
    # Record time
}
```

---

## Network Testing

### Simulate Slow Network

Edit browser context:
```csharp
var context = await browser.NewContextAsync(
    new BrowserNewContextOptions
    {
        IgnoreHTTPSErrors = true
        // Note: Playwright doesn't have built-in throttling
        // Consider using Chrome DevTools Protocol directly
    }
);
```

### Test with Network Issues

Try these URLs:
```
https://httpstat.us/200?sleep=5000   (5 second delay)
https://httpstat.us/500               (Server error)
https://httpstat.us/404               (Not found)
```

---

## Log Analysis

### Enable Debug Logging

Edit `appsettings.Development.json`:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Debug"
    }
  }
}
```

### View Logs

Logs appear in terminal during `dotnet run --project PerfAnalyzer.csproj`:
```
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 POST /Analyzer/Analyze
```

---

## Automated Testing (Example)

```csharp
[TestClass]
public class PerformanceAnalyzerTests
{
    [TestMethod]
    public async Task AnalyzeExampleCom_ReturnsMetrics()
    {
        // Arrange
        var service = new PerformanceAnalysisService();
        
        // Act
        var report = await service.AnalyzePerformanceAsync(
            "https://example.com"
        );
        
        // Assert
        Assert.IsNotNull(report);
        Assert.IsTrue(report.TotalLoadTime > 0);
        Assert.IsTrue(report.TotalRequests > 0);
        Assert.IsNull(report.ErrorMessage);
    }
}
```

---

## Performance Optimization Tips

### For Faster Analysis
1. Analyze simple sites first (example.com)
2. Use wired network (faster than WiFi)
3. Close other applications
4. Run during off-peak hours

### For Better Accuracy
1. Analyze multiple times
2. Average the results
3. Note network conditions
4. Consider caching

---

## When to Restart

**Restart after:**
- Adding NuGet packages
- Changing configuration files
- Modifying Program.cs
- Port conflicts
- High memory usage

**No restart needed:**
- Changing view files (.cshtml)
- Modifying CSS
- Adding comments
- JavaScript changes (if any)

---

## Getting Help

### Check These Files
1. `README.md` - Overview
2. `QUICKSTART.md` - Quick reference
3. `API.md` - API documentation
4. `TECHNICAL.md` - Architecture details
5. This file - Troubleshooting

### Debug Steps
1. Run `dotnet clean PerfAnalyzer.csproj && dotnet build PerfAnalyzer.csproj`
2. Check .NET version: `dotnet --version`
3. Verify port: `netstat -ano | findstr :5000`
4. Check Playwright: `playwright --version`
5. Try different URL
6. Enable debug logging
7. Check browser console (F12)

---

## Performance Baseline

Expected metrics for various sites:

| Site | Load Time | Size | Requests |
|------|-----------|------|----------|
| example.com | 0.5-1.5s | 1-2MB | 5-10 |
| wikipedia.org | 2-4s | 5-10MB | 20-40 |
| github.com | 3-8s | 10-20MB | 50-100 |

Your results will vary based on:
- Internet speed
- Server performance
- Network latency
- Browser startup time

---

## Quick Command Reference

```powershell
# Start development
dotnet run --project PerfAnalyzer.csproj

# Build only
dotnet build PerfAnalyzer.csproj

# Clean build
dotnet clean PerfAnalyzer.csproj; dotnet build PerfAnalyzer.csproj

# Run with specific port
dotnet run --project PerfAnalyzer.csproj --launch-profile http

# Check Playwright
playwright --version

# Kill process on port
Stop-Process -Id <PID> -Force

# Find process on port
Get-NetTCPConnection -LocalPort 5000
```

---

**Last Updated:** March 24, 2026
