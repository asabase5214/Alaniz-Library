# Google Sheets Setup Guide

This library app can sync with Google Sheets to maintain your book collection. Due to browser CORS restrictions, there are two ways to set this up:

## Option 1: Google Apps Script (Recommended - No CORS Issues)

This method creates a web app that bypasses CORS restrictions and works reliably.

### Steps:

1. **Open your Google Sheet** with your book data
   - The first row should contain headers (column names)
   - **Required headers:** `id`, `title`
   - **Recommended headers:** `isbn`, `quantity`, `description`, `authors`, `publisher`, `published`, `pages`, `language`, `location`
   - **Checkout columns (auto-created if missing):** `checkedOutByName`, `checkedOutByEmail`, `checkedOutDate`
   - The `id` column should contain unique identifiers for each book
   - The checkout columns will be automatically added by the script when someone checks out a book
   - Headers are case-insensitive

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
   
   function doPost(e) {
     try {
       var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
       var data = sheet.getDataRange().getValues();
       var headers = data[0];
       
       // Parse the incoming JSON
       var params = JSON.parse(e.postData.contents);
       var bookId = params.id;
       var action = params.action; // 'checkout' or 'return'
       
       // Find the book row by ID
       var idIndex = headers.indexOf('id');
       if (idIndex === -1) {
         return ContentService
           .createTextOutput(JSON.stringify({error: 'ID column not found'}))
           .setMimeType(ContentService.MimeType.JSON);
       }
       
       // Find indices for checkout columns (create if they don't exist)
       var checkedOutByNameIndex = headers.indexOf('checkedOutByName');
       var checkedOutByEmailIndex = headers.indexOf('checkedOutByEmail');
       var checkedOutDateIndex = headers.indexOf('checkedOutDate');
       
       // Add new columns if they don't exist
       if (checkedOutByNameIndex === -1) {
         headers.push('checkedOutByName');
         checkedOutByNameIndex = headers.length - 1;
         sheet.getRange(1, checkedOutByNameIndex + 1).setValue('checkedOutByName');
       }
       if (checkedOutByEmailIndex === -1) {
         headers.push('checkedOutByEmail');
         checkedOutByEmailIndex = headers.length - 1;
         sheet.getRange(1, checkedOutByEmailIndex + 1).setValue('checkedOutByEmail');
       }
       if (checkedOutDateIndex === -1) {
         headers.push('checkedOutDate');
         checkedOutDateIndex = headers.length - 1;
         sheet.getRange(1, checkedOutDateIndex + 1).setValue('checkedOutDate');
       }
       
       // Find the book row
       var rowIndex = -1;
       for (var i = 1; i < data.length; i++) {
         if (data[i][idIndex] == bookId) {
           rowIndex = i + 1; // +1 because sheet rows are 1-indexed
           break;
         }
       }
       
       if (rowIndex === -1) {
         return ContentService
           .createTextOutput(JSON.stringify({error: 'Book not found'}))
           .setMimeType(ContentService.MimeType.JSON);
       }
       
       // Update the checkout status
       if (action === 'checkout') {
         sheet.getRange(rowIndex, checkedOutByNameIndex + 1).setValue(params.byName || '');
         sheet.getRange(rowIndex, checkedOutByEmailIndex + 1).setValue(params.byEmail || '');
         // Use provided date or generate current timestamp if not provided
         sheet.getRange(rowIndex, checkedOutDateIndex + 1).setValue(params.date || new Date().toISOString());
       } else if (action === 'return') {
         // Clear all checkout fields on return
         sheet.getRange(rowIndex, checkedOutByNameIndex + 1).setValue('');
         sheet.getRange(rowIndex, checkedOutByEmailIndex + 1).setValue('');
         sheet.getRange(rowIndex, checkedOutDateIndex + 1).setValue('');
       }
       
       return ContentService
         .createTextOutput(JSON.stringify({success: true}))
         .setMimeType(ContentService.MimeType.JSON);
     } catch (error) {
       return ContentService
         .createTextOutput(JSON.stringify({error: error.toString()}))
         .setMimeType(ContentService.MimeType.JSON);
     }
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

**Important:** Checkout/return functionality will NOT sync to Google Sheets when using the CSV method. This is because CSV URLs are read-only. For shared checkout/return status across all users, you must use Option 1 (Google Apps Script).

---

## Option 3: Local Storage Only

If you don't want to use Google Sheets at all:

1. Open `index.html`
2. Find the line with `const GOOGLE_SHEET_URL = '...'`
3. Set it to an empty string: `const GOOGLE_SHEET_URL = '';`
4. The app will work entirely with localStorage

**Note:** With this option, checkout/return status will only be stored locally on each user's device and will not be shared across users.

---

## How the Sync Works

- **On page load:** The app immediately loads from localStorage (cached data)
- **In background:** The app tries to sync with Google Sheets
- **If sync succeeds:** Data is updated and saved to localStorage
- **If sync fails:** App continues working with cached data
- **On checkout/return:** The app updates Google Sheets immediately via POST request and also saves to localStorage
- **Sharing checkout status:** When you check out or return a book, it's written to Google Sheets so all users see the same status on their next page load or sync

This means the app works **offline** and continues functioning even if Google Sheets is unavailable, but checkout/return status will only be shared when Google Sheets sync is working.

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

### Checkout/return status not shared across users

If checkout/return changes aren't visible to other users:

1. **Verify you're using Google Apps Script (Option 1):** Checkout/return sync only works with Google Apps Script URLs, not published CSV URLs
2. **Check the console for errors:** Look for messages like "Sent checkout update to Google Sheets"
3. **Ensure your Google Sheet has an `id` column:** The Apps Script needs book IDs to update the correct rows
4. **Have other users refresh the page:** Changes sync on page load
5. **Check the checkout columns in your Google Sheet:** Verify that `checkedOutByName`, `checkedOutByEmail`, and `checkedOutDate` columns are being updated

If using a published CSV URL (Option 2), checkout/return will only work locally and won't sync.

---

## Data Format

Your Google Sheet should have these columns (case-insensitive):

| id | isbn | title | quantity | description | authors | publisher | published | pages | language | location | checkedOutByName | checkedOutByEmail | checkedOutDate |
|----|------|-------|----------|-------------|---------|-----------|-----------|-------|----------|----------|------------------|-------------------|----------------|
| b123abc | 9780132350884 | Clean Code | 1 | A handbook... | Robert C. Martin | Prentice Hall | 2008-08-01 | 464 | English | Shelf A | | | |

**Required columns:** `id`, `title`.
**Checkout columns:** `checkedOutByName`, `checkedOutByEmail`, `checkedOutDate` (these will be automatically added by the Apps Script if they don't exist).

The `id` column must contain unique identifiers for each book. When you check out or return books through the app, the checkout columns will be automatically updated in Google Sheets, making the status visible to all users.
