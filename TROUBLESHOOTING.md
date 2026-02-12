# Troubleshooting Guide

This guide helps you fix common issues with the Alaniz Library app.

---

## Problem: All Books Show "Untitled" After Deploying Apps Script

### Symptoms
- You deployed the Google Apps Script
- You updated `GOOGLE_SHEET_URL` in `index.html` with your Apps Script URL
- All books in your library now show "Untitled" instead of their actual titles
- Book data seems to be missing or not loading correctly

### Root Cause
**This was a bug in the app (now fixed in this version).** The app was not normalizing field names from the JSON data returned by Google Apps Script. If your Google Sheet has headers like "Title" (capital T) or "Authors", Apps Script returns those exact names, but the app expected lowercase "title" and "authors".

### Solution

**If you just pulled the latest code:** The bug is fixed! Just refresh your browser.

**Step 1: Make sure you have the latest code**

1. Pull the latest version of `index.html` from this repository
2. The fix normalizes field names from Apps Script JSON data

**Step 2: Clear Cache and Refresh**

1. Open your browser's Developer Console (F12 or Right-click → Inspect)
2. Clear your browser cache or do a hard refresh:
   - Windows/Linux: `Ctrl + Shift + R` or `Ctrl + F5`
   - Mac: `Cmd + Shift + R`

**Step 3: Verify It's Working**

Open the browser console and look for these messages:
- ✅ GOOD: `✓ Parsed JSON data from Google Apps Script`
- ❌ BAD: `⚠️ Using CSV format from published sheet`

If you see the warning, you haven't updated the URL correctly yet.

---

## Problem: Checking Out a Book Doesn't Sync to Other Devices

### Symptoms
- You check out a book on one device
- Other users/devices don't see the checkout status
- The book appears available on other devices even though you checked it out

### Root Cause
The app is using a CSV URL instead of a Google Apps Script URL. CSV URLs are read-only and cannot receive POST requests to update checkout status.

### Solution

**You need to use a Google Apps Script URL, not a CSV URL.**

**Step 1: Find Your Apps Script URL**

1. Open your Google Sheet
2. Go to **Extensions** → **Apps Script**
3. Click **Deploy** → **Manage deployments**
4. Copy the **Web app URL** (it should look like: `https://script.google.com/macros/s/AKfycby.../exec`)
5. If you don't have a deployment yet, see `GOOGLE_SHEETS_SETUP.md` for complete setup instructions

**Step 2: Update index.html**

1. Open `index.html` in a text editor
2. Find this line (in the configuration section near the end of the file):
   ```javascript
   const GOOGLE_SHEET_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-...';
   ```

3. Replace the entire URL with your Apps Script URL:
   ```javascript
   const GOOGLE_SHEET_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';
   ```

4. Save the file and refresh your browser

### How to Know If It's Fixed

After updating the URL:

1. Open the browser console (F12)
2. Check out a book
3. Look for this message:
   - ✅ GOOD: `✓ Sent checkout update to Google Sheets for book b...`
   - ❌ BAD: `⚠️ CHECKOUT SYNC NOT AVAILABLE` (followed by warnings)

If you see the warning, you're still using a CSV URL.

### Verify in Google Sheets

1. Open your Google Sheet
2. Check out a book in the app
3. Look for the columns: `checkedOutByName`, `checkedOutByEmail`, `checkedOutDate`
4. These columns should be automatically created and populated
5. Refresh the app on another device - the checkout status should appear there too

---

## Quick Reference: CSV vs Apps Script URLs

| Type | Example | Can Read Books? | Can Sync Checkouts? |
|------|---------|-----------------|---------------------|
| **CSV (Published)** | `https://docs.google.com/spreadsheets/d/e/2PACX-...` | ⚠️ Maybe (unreliable) | ❌ NO |
| **Apps Script** | `https://script.google.com/macros/s/.../exec` | ✅ YES | ✅ YES |

**Always use Apps Script URLs for full functionality!**

---

## Console Messages Explained

### Good Messages (Everything is working)

```
✓ Parsed JSON data from Google Apps Script
✓ Successfully synced 42 books from Google Sheets
✓ Sent checkout update to Google Sheets for book b1a2b3c4
```

### Warning Messages (Configuration issue)

```
⚠️ CONFIGURATION ISSUE DETECTED
⚠️ Using CSV format from published sheet
⚠️ CHECKOUT SYNC NOT AVAILABLE
```

**What to do:** Update your `GOOGLE_SHEET_URL` to use Apps Script instead of CSV.

### Error Messages (Network or permissions issue)

```
Could not sync with Google Sheets: HTTP error! status: 403
```

**What to do:** 
- Make sure your Apps Script is deployed with "Who has access: **Anyone**"
- Re-authorize the script if needed
- Check that the URL is correct

---

## Still Having Issues?

### Problem: Headers not mapping correctly

If your books still show "Untitled" even after switching to Apps Script:

1. Check your Google Sheet headers (first row)
2. Make sure you have these required columns (case-insensitive):
   - `id` - Required, unique identifier
   - `title` - Required, book title
3. The Apps Script should return these exact field names

### Problem: Apps Script not deployed correctly

Error: `HTTP error! status: 404`

**Solution:**
1. In Apps Script, click **Deploy** → **New deployment**
2. Select type: **Web app**
3. Execute as: **Me**
4. Who has access: **Anyone**
5. Click **Deploy**
6. Authorize when prompted
7. Copy the new URL and update `index.html`

### Problem: Permission denied

Error: `HTTP error! status: 403`

**Solution:**
1. In Apps Script deployment settings, verify "Who has access: **Anyone**"
2. Re-authorize the script:
   - Click **Run** in Apps Script editor
   - Grant permissions when prompted
3. Re-deploy if needed

---

## Need More Help?

1. Open your browser's console (F12)
2. Look for detailed warning messages
3. Follow the instructions in the console warnings
4. See `GOOGLE_SHEETS_SETUP.md` for complete setup instructions
