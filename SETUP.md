# Wedding Site Deployment Guide

## 1. Google Apps Script Backend (RSVP + Wishes)

### Create the Google Sheet
1. Go to https://sheets.google.com → New spreadsheet
2. Name it "Wedding RSVP & Wishes"
3. Create two sheets (tabs):
   - **RSVP** — columns: `timestamp | name | pax | attending`
   - **Wishes** — columns: `timestamp | name | message`

### Deploy the Apps Script
1. In the spreadsheet: **Extensions → Apps Script**
2. Delete the default code, paste this:

```javascript
var SHEET_ID = SpreadsheetApp.getActiveSpreadsheet().getId();

function doPost(e) {
  var ss = SpreadsheetApp.openById(SHEET_ID);
  var data = e.parameter;

  if (data.type === 'rsvp') {
    var sheet = ss.getSheetByName('RSVP');
    sheet.appendRow([data.timestamp, data.name, data.pax, data.attending]);
  } else if (data.type === 'wish') {
    var sheet = ss.getSheetByName('Wishes');
    sheet.appendRow([data.timestamp, data.name, data.message]);
  }

  return ContentService.createTextOutput(JSON.stringify({result: 'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}

function doGet(e) {
  var ss = SpreadsheetApp.openById(SHEET_ID);

  if (e.parameter.type === 'wishes') {
    var sheet = ss.getSheetByName('Wishes');
    var data = sheet.getDataRange().getValues();
    var rows = [];
    for (var i = 1; i < data.length; i++) {
      rows.push({
        timestamp: data[i][0],
        name: data[i][1],
        message: data[i][2]
      });
    }
    return ContentService.createTextOutput(JSON.stringify(rows))
      .setMimeType(ContentService.MimeType.JSON);
  }

  return ContentService.createTextOutput(JSON.stringify({error: 'unknown type'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Deploy → New deployment**
4. Type: **Web app**
5. Execute as: **Me**
6. Who has access: **Anyone**
7. Click **Deploy**, authorize, copy the URL

### Paste URL in index.html
Open `index.html`, find this line near the bottom:
```javascript
var SCRIPT_URL = '';
```
Paste your Apps Script URL:
```javascript
var SCRIPT_URL = 'https://script.google.com/macros/s/YOUR_ID/exec';
```

## 2. GitHub Repository

```bash
cd wedding
git init
git add index.html SETUP.md
git commit -m "feat: wedding invitation site for Filzah & Nazmirul"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/wedding.git
git push -u origin main
```

## 3. Cloudflare Pages Deployment

1. Go to https://dash.cloudflare.com → **Workers & Pages** → **Create**
2. Choose **Pages** → **Connect to Git**
3. Select your `wedding` repository
4. Build settings:
   - Framework preset: **None**
   - Build command: (leave empty)
   - Build output directory: `.` (or `/`)
5. Click **Save and Deploy**

### Custom Domain
1. After deployment, go to your Pages project → **Custom domains**
2. Add: `nazandfilzah.nzhomelab.online`
3. Cloudflare auto-creates the CNAME record since DNS is already on Cloudflare
4. Wait ~1 min for SSL certificate

Your site will be live at: **https://nazandfilzah.nzhomelab.online**

## 4. Google Drive Video Setup

Make sure your video is shared:
1. Open https://drive.google.com
2. Right-click the video → **Share**
3. Set to **Anyone with the link** → **Viewer**
4. The embed URL in index.html is already set to your file ID

## Summary

| What | Where |
|------|-------|
| Site hosting | Cloudflare Pages (free) |
| Custom domain | nazandfilzah.nzhomelab.online |
| Video | Google Drive (your existing upload) |
| RSVP data | Google Sheets |
| Wishes data | Google Sheets |
| Cost | $0 |
