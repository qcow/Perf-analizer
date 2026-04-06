# 🚀 MonsterASP.net Deployment Guide

## ❌ 403 Forbidden Error - SOLVED

If you're getting **403 - Forbidden: Access is denied** error, follow these steps:

---

## ✅ Changes Made to Fix This

### 1. **Updated Program.cs**
- Changed default route from `Analyzer` to `Home`
- Added explicit route for `/analyzer` access
- Now MonsterASP.net can find the home page

### 2. **Added HomeController.cs**
- MonsterASP.net requires a Home controller
- Automatically redirects to Analyzer
- Provides standard error handling

### 3. **Added Home Views**
- Created `Views/Home/Index.cshtml`
- Serves as landing page
- Has link to Analyzer

---

## 📋 MonsterASP.net Requirements Checklist

Before uploading, verify:

- [ ] **Project builds successfully**
  ```powershell
  dotnet build PerfAnalyzer.csproj -c Release
  ```

- [ ] **No compilation errors**
  Should show: `Build succeeded`

- [ ] **Has Home controller**
  Check: `Controllers/HomeController.cs` exists

- [ ] **Has Home views**
  Check: `Views/Home/Index.cshtml` exists

- [ ] **Web.config exists**
  Check: File in root folder

- [ ] **appsettings.json correct**
  Has: `"AllowedHosts": "*"`

---

## 🚀 Deployment Steps

### Step 1: Build for Release
```powershell
cd "d:\Research\PERF-ANALYZER - UPGRADE"
dotnet build PerfAnalyzer.csproj -c Release
```

### Step 2: Publish
```powershell
dotnet publish PerfAnalyzer.csproj -c Release -o ./publish
```

Output will be in `publish` folder.

### Step 3: Upload to MonsterASP.net

Using FTP or File Manager:

1. Connect to your hosting account
2. Navigate to `wwwroot` folder
3. Upload ALL files from `publish` folder
4. Ensure `appsettings.json` is uploaded

### Step 4: Test in Browser
```
https://yourdomainname.com
```

Should show the home page with link to Analyzer.

---

## 🔐 MonsterASP.net Specific Settings

### In your hosting control panel:

1. **Runtime Version:** .NET 8.0 (if available) or latest
2. **Application Pool:** Default
3. **Permissions:** 
   - Read: ✅ Yes
   - Write: ✅ Yes
   - Execute: ✅ Yes

### In launchSettings.json (local testing):
```json
{
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

---

## 📁 File Structure to Upload

The `publish` folder should contain:

```
publish/
├── appsettings.json           ← IMPORTANT
├── appsettings.Development.json
├── PerfAnalyzer.dll           ← Main app DLL
├── PerfAnalyzer.runtimeconfig.json
├── Views/
│   ├── Analyzer/
│   │   └── Index.cshtml
│   ├── Home/
│   │   └── Index.cshtml
│   └── Shared/
│       ├── _Layout.cshtml
│       └── ...
├── wwwroot/
│   ├── css/
│   ├── js/
│   ├── lib/
│   └── favicon.ico
└── [Other .NET runtime files]
```

---

## 🐛 Troubleshooting

### Still getting 403 error?

**Check 1: Home Controller**
```csharp
// Should exist in Controllers/HomeController.cs
public class HomeController : Controller
{
    public IActionResult Index() { ... }
}
```

**Check 2: Routes in Program.cs**
```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Check 3: Permissions**
- Ask MonsterASP.net support to check folder permissions
- Ensure `wwwroot` is writable (might be needed for logs)

**Check 4: HTTPS Redirect**
If you get SSL errors, try:
```csharp
// Temporarily disable HTTPS redirect
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // Comment out next line if SSL issues
    // app.UseHsts();
    // app.UseHttpsRedirection();
}
```

### Getting 500 errors instead?

Check the error log:
1. Access MonsterASP.net control panel
2. Look for error logs in `App_Data` or `Logs` folder
3. Check for `.log` files

Common 500 error causes:
- Wrong .NET runtime version
- Missing `appsettings.json`
- Playwright not available (not critical for this)
- Missing views

---

## ✅ Verification Steps

### Local Testing Before Upload

```powershell
# 1. Build
dotnet build PerfAnalyzer.csproj -c Release

# 2. Run locally
dotnet run --project PerfAnalyzer.csproj

# 3. Test URLs:
# http://localhost:5000                    ← Should show home
# http://localhost:5000/Analyzer/Index     ← Should show analyzer
```

### After Uploading

1. Visit: `https://yourdomain.com`
   - Should show home page
   
2. Click "Go to Analyzer" link
   - Should show analyzer page
   
3. Test analysis:
   - Enter URL: `https://example.com`
   - Click "Analyze"
   - Should work

---

## 📞 MonsterASP.net Support Info

If issues persist, contact MonsterASP.net support and provide:

1. **Error message:** 403 Forbidden
2. **URL tried:** https://yourdomain.com
3. **App info:** ASP.NET Core 8 MVC
4. **Default document:** Set to `index.html` or blank (auto-route)
5. **Ask them to:**
   - Verify .NET Core support enabled
   - Check file permissions in wwwroot
   - Review application pool settings

---

## 🎯 Quick Summary

**What was wrong:**
- App defaulted to `/Analyzer/Index` instead of `/Home/Index`
- MonsterASP.net couldn't find a home page

**What was fixed:**
- Added HomeController that redirects to Analyzer
- Updated default route to start at Home
- Created Home views

**Result:**
- ✅ 403 error should be gone
- ✅ Site now accessible
- ✅ Can navigate to Analyzer

---

**Try deploying now and let me know if it works!** 🚀

If you still get 403, I can help with additional troubleshooting.
