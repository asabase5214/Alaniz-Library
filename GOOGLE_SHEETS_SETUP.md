# Google Sheets Setup Guide

This library app can sync with Google Sheets to maintain your book collection. Due to browser CORS restrictions, there are two ways to set this up:

## Option 1: Google Apps Script (Recommended - No CORS Issues)

This method creates a web app that bypasses CORS restrictions and works reliably.

### Steps:

1. **Open your Google Sheet** with your book data
   - The first row should contain headers: `isbn`, `title`, `quantity`, `description`, `authors`, `publisher`, `published`, `pages`, `language`, `location`

2. **Create an Apps Script:**
   - In your Google Sheet, go to **Extensions** → **Apps Script**
   
3. **Add this code:**
   ```javascript
   function doGet(e) {
     var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
     var data = sheet.getDataRange().getValues();
     
     var headers = data[0];
     var books = [];
     
     for (var i = 1; i < data.length; i++) {
       var book = {};
       for (var j = 0; j < headers.length; j++) {
         book[headers[j]] = data[i][j];
       }
       books.push(book);
     }
     
     return ContentService
       .createTextOutput(JSON.stringify(books))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```

4. **Deploy the script:**
   - Click **Deploy** → **New deployment**
   - Click the gear icon ⚙️ next to "Select type"
   - Choose **Web app**
   - Set **Execute as**: Me (your email)
   - Set **Who has access**: Anyone
   - Click **Deploy**
   - **Authorize** the script when prompted
   - Copy the **Web app URL** (looks like: `https://script.google.com/macros/s/...../exec`)

5. **Update the library app:**
   - Open `index.html`
   - Find the line with `const GOOGLE_SHEET_URL = '...'`
   - Replace it with your Web app URL
   - Save the file

### Benefits:
- ✅ No CORS issues
- ✅ Returns clean JSON data
- ✅ Works reliably across all browsers
- ✅ Can add authentication later if needed

---

## Option 2: Published CSV (May have CORS issues)

This method is simpler but may not work in all browsers due to CORS restrictions.

### Steps:

1. **Open your Google Sheet**

2. **Publish to web:**
   - Go to **File** → **Share** → **Publish to web**
   - Under "Link", select your specific sheet tab
   - Change format to **Comma-separated values (.csv)**
   - Click **Publish**
   - Copy the generated URL

3. **Update the library app:**
   - Open `index.html`
   - Find the line with `const GOOGLE_SHEET_URL = '...'`
   - Replace it with your published CSV URL
   - Save the file

### Note:
This method may be blocked by browser CORS policies. If you see "Failed to fetch" in the console, use Option 1 instead.

---

## Option 3: Local Storage Only

If you don't want to use Google Sheets at all:

1. Open `index.html`
2. Find the line with `const GOOGLE_SHEET_URL = '...'`
3. Set it to an empty string: `const GOOGLE_SHEET_URL = '';`
4. The app will work entirely with localStorage

---

## How the Sync Works

- **On page load:** The app immediately loads from localStorage (cached data)
- **In background:** The app tries to sync with Google Sheets
- **If sync succeeds:** Data is updated and saved to localStorage
- **If sync fails:** App continues working with cached data

This means the app works **offline** and continues functioning even if Google Sheets is unavailable.

---

## Troubleshooting

### "Could not sync with Google Sheets" in console

This is normal if:
- You haven't set up a Google Sheets URL yet
- The Google Sheets URL is blocked by CORS
- You're offline

**Solution:** Use Option 1 (Google Apps Script) for reliable syncing.

### Changes in Google Sheets aren't showing up

1. Refresh the page (app syncs on every page load)
2. Check browser console for sync messages
3. If using Apps Script, re-deploy the script
4. Clear browser cache and try again

### App shows "No books found"

This means both:
- localStorage is empty (no cached data)
- Google Sheets sync failed or returned no data

**Solution:** 
- Import sample data using the "Import Sample" button
- Or manually add books using "+ Add Book"
- Or fix the Google Sheets sync using Option 1

---

## Data Format

Your Google Sheet should have these columns (case-insensitive):

| isbn | title | quantity | description | authors | publisher | published | pages | language | location |
|------|-------|----------|-------------|---------|-----------|-----------|-------|----------|----------|
| 9780132350884 | Clean Code | 1 | A handbook... | Robert C. Martin | Prentice Hall | 2008-08-01 | 464 | English | Shelf A |

Not all columns are required, but `title` is recommended for best experience.
