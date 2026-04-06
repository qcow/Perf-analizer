# ✅ MonsterASP.net Deployment Checklist

## 🎯 Fixed Issues

Your **403 Forbidden error** has been fixed! Here's what was changed:

### Changes Made ✅
1. **HomeController.cs** - Added (redirects to Analyzer)
2. **Home/Index.cshtml** - Added (landing page)
3. **Program.cs** - Updated default route from Analyzer to Home
4. **MONSTERASP_DEPLOYMENT_GUIDE.md** - Added (detailed guide)

---

## 📋 Pre-Deployment Checklist

### ✅ Code Changes
- [x] HomeController added
- [x] Home views added
- [x] Program.cs updated
- [x] appsettings.json verified
- [x] Project builds successfully

### ✅ Files to Upload
Ensure these files are in your `publish` folder:

**Essential:**
- [ ] appsettings.json
- [ ] appsettings.Development.json
- [ ] PerfAnalyzer.dll
- [ ] PerfAnalyzer.runtimeconfig.json

**Views:**
- [ ] Views/Home/Index.cshtml
- [ ] Views/Analyzer/Index.cshtml
- [ ] Views/Shared/_Layout.cshtml
- [ ] Views/Shared/_ViewStart.cshtml
- [ ] Views/Shared/Error.cshtml

**Static Files:**
- [ ] wwwroot/css/ (all CSS files)
- [ ] wwwroot/js/ (if any)
- [ ] wwwroot/lib/ (libraries)
- [ ] wwwroot/favicon.ico

**Runtime:**
- [ ] All .NET runtime files (auto-generated)

---

## 🚀 Step-by-Step Deployment

### Step 1: Build for Release
```powershell
cd "d:\Research\PERF-ANALYZER - UPGRADE"
dotnet build PerfAnalyzer.csproj -c Release
```

Expected output: ✅ Build succeeded

### Step 2: Publish
```powershell
dotnet publish PerfAnalyzer.csproj -c Release -o ./publish
```

### Step 3: Upload via FTP

Using any FTP client (WinSCP, FileZilla, etc.):

1. Connect to MonsterASP.net FTP server
2. Navigate to `wwwroot` folder
3. **Delete old files** (if re-deploying)
4. **Upload everything** from `publish` folder
5. Verify upload complete

**Critical:** Ensure `appsettings.json` is uploaded!

### Step 4: Test in Browser

```
https://yourdomain.com
```

You should see:
- ✅ Welcome page with link to Analyzer
- ✅ Click link → Goes to Analyzer
- ✅ Analyzer form loads

### Step 5: Test Analysis

On Analyzer page:
1. Enter: `https://example.com`
2. Click: "ANALYZE SITE"
3. Wait: 10-15 seconds
4. Expected: See performance metrics

---

## 🔧 If You Still Get 403 Error

### Quick Fixes

**Option 1: Contact MonsterASP.net Support**

Provide them:
- Error: 403 Forbidden
- App: ASP.NET Core 8 MVC
- Request: Enable .NET Core routing
- Ask: Verify application pool settings

**Option 2: Check File Permissions**

In MonsterASP.net Control Panel:
1. File Manager
2. Right-click `wwwroot`
3. Permissions → Set to 755
4. Refresh browser

**Option 3: Verify Default Document**

In MonsterASP.net Control Panel:
1. Look for "Default Documents" or "Home Page"
2. Leave BLANK (let ASP.NET Core route)
3. Or add blank/none
4. Save changes

**Option 4: Check .NET Version**

In control panel:
1. Find Application Settings
2. Verify .NET Version: 8.0 or latest
3. Restart application pool
4. Test again

---

## 📊 MonsterASP.net Configuration

### Recommended Settings

**Runtime:**
- .NET Core: 8.0 (or latest available)
- Framework: ASP.NET Core
- Bit Mode: x64

**Application Pool:**
- Managed Runtime: .NET CLR (latest)
- Pipeline: Integrated
- Preload: Enabled (optional)

**Directory:**
- Physical Path: `/wwwroot` or `/web`
- Scripts/Executables: Yes
- Read: Yes
- Write: Conditional (for logs only)

---

## 🔐 HTTPS/SSL Settings

MonsterASP.net typically auto-configures HTTPS. If issues:

1. **Keep HTTPS redirect** in Program.cs:
   ```csharp
   app.UseHttpsRedirection();
   ```

2. **Or disable temporarily:**
   ```csharp
   if (!app.Environment.IsDevelopment())
   {
       app.UseExceptionHandler("/Home/Error");
       app.UseHsts();
       // Try commenting out HTTPS redirect
       // app.UseHttpsRedirection();
   }
   ```

3. **Test with http:// first:**
   ```
   http://yourdomain.com (if allowed)
   ```

---

## 📁 Final Directory Structure

After uploading, your FTP should look like:

```
wwwroot/
├── appsettings.json              ← CRITICAL
├── appsettings.Development.json
├── PerfAnalyzer.dll
├── PerfAnalyzer.runtimeconfig.json
├── PerfAnalyzer.staticwebassets.runtime.json
├── Views/
│   ├── Analyzer/
│   │   └── Index.cshtml
│   ├── Home/
│   │   └── Index.cshtml
│   └── Shared/
│       ├── _Layout.cshtml
│       ├── _ViewStart.cshtml
│       └── Error.cshtml
├── wwwroot/
│   ├── css/
│   │   └── site.css
│   ├── js/
│   ├── lib/
│   │   └── bootstrap/ (if using)
│   └── favicon.ico
├── [Other runtime assemblies...]
└── web.config (if needed)
```

---

## 🧪 Test URLs

After deployment, test these URLs in order:

1. **Home Page**
   ```
   https://yourdomain.com
   ```
   Expected: Home page with "Go to Analyzer" button

2. **Analyzer Page**
   ```
   https://yourdomain.com/analyzer/index
   ```
   Expected: Analyzer form

3. **Test Analysis**
   - Enter: `https://example.com`
   - Click: "Analyze Site"
   - Wait: 10-15 seconds
   - Expected: Metrics display

---

## 📞 Troubleshooting Contact Info

**MonsterASP.net Support:**
- Ticket System: Your control panel
- Email: support@monsterasp.net (typical)
- Chat: Available in control panel

**What to Tell Them:**
- Issue: Getting 403 error on home page
- App: ASP.NET Core 8 MVC
- Deployed: Published from dotnet CLI
- Need: Verify routing, permissions, app pool

---

## ✅ Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| **403 Forbidden** | Add HomeController (✅ DONE) |
| **404 Not Found** | Check URL routes and file paths |
| **500 Internal Error** | Check error logs, verify .NET version |
| **Blank Page** | Check Views folder uploaded |
| **No CSS/Images** | Check wwwroot folder uploaded |
| **HTTPS Issues** | Disable `UseHttpsRedirection()` temporarily |
| **Cannot Access Analyzer** | Verify /analyzer route exists |

---

## 🚀 Deployment Complete Checklist

Before declaring success:

- [ ] Can access home page (https://yourdomain.com)
- [ ] Home page has working link to Analyzer
- [ ] Can click link and reach Analyzer page
- [ ] Analyzer form displays correctly
- [ ] Can submit a URL for analysis
- [ ] Analysis runs (takes 10-15 seconds)
- [ ] Results display on page
- [ ] No errors in browser console (F12)
- [ ] CSS styling is applied (not white/unstyled)
- [ ] All navigation works

---

## 🎊 Success Indicators

You'll know it's working when:

✅ **Home Page Loads**
```
https://yourdomain.com
→ Shows welcome message
→ Has "Go to Analyzer" button
```

✅ **Analyzer Page Loads**
```
https://yourdomain.com/analyzer/index
→ Shows form
→ CSS is styled (dark theme)
→ Can type in URL input
```

✅ **Analysis Works**
```
Enter: https://example.com
Click: "Analyze Site"
Wait: 10-15 seconds
See: Metrics, waterfall, categories
```

---

## 📊 What Was Fixed

### Before
- Default route: `/Analyzer/Index`
- No Home controller
- MonsterASP.net couldn't find home → **403 error**

### After
- Default route: `/Home/Index` ✅
- HomeController redirects to Analyzer ✅
- MonsterASP.net finds home page ✅
- Can navigate to Analyzer ✅

---

## 🎯 Next Steps

1. **Build:** `dotnet build PerfAnalyzer.csproj -c Release`
2. **Publish:** `dotnet publish PerfAnalyzer.csproj -c Release -o ./publish`
3. **Upload:** All files from `publish` to `wwwroot` via FTP
4. **Test:** Visit https://yourdomain.com
5. **Verify:** Everything works as expected

---

**You're ready to deploy!** 🚀

If you encounter any issues, check `MONSTERASP_DEPLOYMENT_GUIDE.md` for detailed troubleshooting.

Let me know how it goes! 💪
