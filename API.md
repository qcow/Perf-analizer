# API Documentation - ROBO PERF-ANALYZER

## Base URL
```
http://localhost:5000
https://localhost:44347
```

---

## Endpoints

### GET /Analyzer/Index
Returns the main analyzer interface.

**Response:** HTML page with analyzer form

**Example:**
```
GET /Analyzer/Index
```

---

### POST /Analyzer/Analyze
Analyzes a website and returns performance metrics.

**Method:** POST  
**URL:** `/Analyzer/Analyze`  
**Content-Type:** `application/x-www-form-urlencoded`

#### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| url | string | Yes | Website URL to analyze (http/https) |

#### Request Example
```html
<form method="POST" action="/Analyzer/Analyze">
  <input type="url" name="url" value="https://example.com" />
  <button type="submit">ANALYZE SITE</button>
</form>
```

#### Response Model

```csharp
PerformanceReport {
  Url: string,                          // Input URL
  IsAnalyzing: bool,                    // Always false on response
  ErrorMessage: string | null,          // Error if analysis failed
  
  // Overall Metrics
  TotalLoadTime: double (ms),           // Total time to load
  TotalSize: long (bytes),              // Sum of all resource sizes
  TotalRequests: int,                   // Number of HTTP requests
  
  // Category Breakdowns
  Scripts: CategoryMetrics,             // JS files
  Styles: CategoryMetrics,              // CSS files
  Images: CategoryMetrics,              // Image files
  Other: CategoryMetrics,               // Other resources
  
  // Detailed Assets
  Assets: AssetMetric[]                 // Individual resources
}
```

#### CategoryMetrics
```csharp
CategoryMetrics {
  TotalSize: long (bytes),
  TotalDuration: double (ms),
  Count: int,
  IsCritical: bool,
  
  // Calculated Properties
  FormattedSize: string (e.g., "450.25 KB"),
  FormattedDuration: string (e.g., "650.50ms")
}
```

#### AssetMetric
```csharp
AssetMetric {
  FileName: string,                     // e.g., "jquery.min.js"
  ResourceType: string,                 // "Script", "Stylesheet", "Image", "Other"
  Size: long (bytes),
  ResponseTime: double (ms),
  StatusCode: int,                      // HTTP status (200, 404, etc)
  IsCritical: bool,                     // True if CRITICAL condition met
  
  // Calculated Properties
  FormattedSize: string,
  FormattedTime: string,
  StatusLabel: string                   // "✓ OK", "⚠️ CRITICAL", "✗ ERROR"
}
```

#### Success Response (HTML)
```html
<!DOCTYPE html>
<html>
  <body>
    <!-- Summary Cards -->
    <div class="metric-card">
      <div class="metric-label">⏱️ Load Time</div>
      <div class="metric-value">1245.50</div>
    </div>
    
    <!-- Category Cards -->
    <div class="category-card critical">
      <div class="category-name">📜 Scripts</div>
      <div class="category-badge critical">⚠️ CRITICAL</div>
    </div>
    
    <!-- Waterfall Table -->
    <table class="asset-table">
      <tr>
        <td>jquery.min.js</td>
        <td><span class="asset-type script">Script</span></td>
        <td>450.25 KB</td>
        <td>650.50ms</td>
        <td><span class="status-critical">⚠️ CRITICAL</span></td>
      </tr>
    </table>
  </body>
</html>
```

#### Error Response (HTML)
```html
<!DOCTYPE html>
<html>
  <body>
    <div class="alert alert-danger">
      <strong>Error:</strong> Invalid URL format
    </div>
  </body>
</html>
```

---

## Critical Conditions

An asset is marked as CRITICAL if:

```
IsCritical = (ResourceType == "Script" AND Size > 300KB)
           OR (ResourceType == "Stylesheet" AND Size > 300KB)
           OR (ResponseTime > 500ms)
```

---

## Error Handling

### Invalid URL Format
```
Input: "not a valid url"
Response: Error message → "Invalid URL format"
```

### Empty URL
```
Input: ""
Response: Error message → "Please enter a valid URL"
```

### Unreachable Site
```
Input: "https://definitely-not-a-real-domain-12345.com"
Response: Error message → "Analysis failed: [detailed error]"
```

### Timeout
```
Input: "https://very-slow-server.example.com"
Response: Error message → "Analysis failed: timeout exceeded"
Timeout: 30 seconds per request
```

---

## Response Codes (Web Page)

| Status | Meaning |
|--------|---------|
| 200 | Analysis completed (view results) |
| 200 | Error occurred (view error message) |
| 500 | Server error |

---

## Examples

### Successful Analysis

**Request:**
```bash
curl -X POST http://localhost:5000/Analyzer/Analyze \
  -d "url=https://example.com"
```

**Response:** HTML page with metrics

```
LOAD TIME:     1245.50ms
TOTAL SIZE:    2.5 MB
REQUESTS:      42

Scripts: ⚠️ CRITICAL (450.25 KB, 650.50ms)
Styles: ✓ OK (85.50 KB, 120.25ms)
Images: ✓ OK (1.8 MB, 450.75ms)
Other: ✓ OK (45.20 KB, 80.50ms)

[Waterfall table with 42 entries...]
```

### Error Response

**Request:**
```bash
curl -X POST http://localhost:5000/Analyzer/Analyze \
  -d "url=invalid-url"
```

**Response:** HTML page with error

```
Error: Invalid URL format
```

---

## Browser Requirements

- Chromium-based (via Playwright)
- Auto-installed on first build
- No user interaction required
- Headless mode (invisible)

---

## Limits

| Item | Limit |
|------|-------|
| Max URL length | 2048 characters |
| Max analysis time | 30 seconds |
| Max resources tracked | Unlimited |
| Concurrent analyses | 1 (sequential) |

---

## Performance Notes

- Analysis takes 5-15 seconds typically
- Depends on target website speed
- Network throttling not applied
- Uses real browser (accurate metrics)

---

## Security Considerations

- No authentication required (local use)
- Ignores HTTPS errors (for testing)
- No data storage (all in-memory)
- No external API calls (except target URL)

---

## Future Enhancements

- [ ] Batch analysis endpoint
- [ ] Custom metrics endpoint
- [ ] Export to JSON/CSV
- [ ] Performance history API
- [ ] Compare multiple sites
- [ ] Network throttling options
- [ ] Custom headers support
- [ ] Cookie management
- [ ] User-Agent customization

---

## Technical Details

### Resource Type Detection
```csharp
.js         → Script
.css        → Stylesheet
.jpg/.png/  → Image
.gif/.webp  → Image
.svg/.ico   → Image
Others      → Other
```

### Size Formatting
```
0 B → 1 B          = "X.XX B"
1 KB → 1024 KB     = "X.XX KB"
1 MB → 1024 MB     = "X.XX MB"
1 GB+              = "X.XX GB"
```

### Time Measurement
- Start: Page.goto() called
- End: WaitUntilState.NetworkIdle achieved
- Unit: Milliseconds (ms)
- Precision: 0.01ms

---

**API Version:** 1.0  
**Last Updated:** March 24, 2026
