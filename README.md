# Shoebox for OneDrive: setup guide

Shoebox is a single web page (`index.html`) that photographs receipts, reads them, asks you the reason for each expense, and keeps everything in your own OneDrive. Setup takes about 20 minutes, once.

## What ends up in your OneDrive

```
OneDrive/
└── Shoebox/
    ├── Shoebox expenses.xlsx      ← your spreadsheet, rebuilt after every change
    ├── ledger.json                ← the master record the app reads and writes
    └── Receipts/
        └── 2026/
            └── 2026-11-03_Cafe-Nero_a1b2c3.jpg
```

The spreadsheet has two sheets. **Expenses** lists date, company, expense type, reason, what was bought, tax, total, currency, payment method, and a "View receipt" link to the photo in OneDrive. **Summary** totals each month by expense type and currency.

`ledger.json` is the source of truth, and the spreadsheet is regenerated from it. Edits you make directly in the spreadsheet will be overwritten the next time you save a receipt, so make changes in the app, or copy the spreadsheet before working in it.

## Step 1: Put the page online (GitHub Pages, free)

Microsoft sign-in only works from a real web address, so the page needs hosting.

1. Create a free account at github.com if you don't have one.
2. Create a new repository, for example `shoebox`. It can be public, since the page contains no personal data or keys.
3. Upload `index.html` to the repository.
4. Go to **Settings → Pages**, set **Source** to "Deploy from a branch", choose `main` and `/ (root)`, and save.
5. After a minute or two your app is live at `https://YOUR-USERNAME.github.io/shoebox/`. Open it; you'll see the "Connect to Microsoft" screen showing this exact address.

Any static host works (Netlify, Cloudflare Pages, Azure Static Web Apps), as long as it uses https.

## Step 2: Register the app with Microsoft (free)

1. Go to portal.azure.com and sign in with your Microsoft account. If asked, create a free Azure account. Registering an app costs nothing.
2. Search for **Microsoft Entra ID**, open it, then **App registrations → New registration**.
3. Name: `Shoebox`.
4. Supported account types: **Accounts in any organizational directory and personal Microsoft accounts**.
5. Redirect URI: choose platform **Single-page application (SPA)** and paste the address shown on the app's setup screen, for example `https://YOUR-USERNAME.github.io/shoebox/`. It must match exactly, including the trailing slash.
6. Click **Register**, then copy the **Application (client) ID**.
7. Open **API permissions → Add a permission → Microsoft Graph → Delegated permissions**, add `Files.ReadWrite`, and save. `User.Read` is usually there already.

Paste the client ID into the app's setup screen and sign in. Microsoft will ask you to approve access to your files once.

If you'd rather not paste it on each new device, put it in `index.html` at the top of the script (`clientId: "..."`) before uploading.

**Work or school accounts:** some organizations require an administrator to approve new apps. If sign-in says admin approval is needed, ask your IT team, or use a personal Microsoft account.

## Step 3 (optional): Automatic receipt reading

Without this, you photograph the receipt and type in the details yourself; the photo and everything else still saves to OneDrive.

1. Go to console.anthropic.com, create an account, add billing, and create an API key.
2. Set a monthly spending limit there so costs can never surprise you. Each receipt is a single small request.
3. In Shoebox, open **Settings (⚙︎)**, paste the key, and choose a model: Haiku 5.5 is faster and cheaper, Sonnet 5.5 copes better with faded or crumpled receipts.

The key is stored only in that phone's browser and is sent only to Anthropic. Anyone with access to your unlocked phone and browser could read it, so keep the spending limit modest. API documentation: https://docs.claude.com

## Step 4: Put it on your Android home screen

Open your Shoebox address in Chrome, tap the **⋮** menu, then **Add to Home screen**. It then opens like an app, and **Take photo** goes straight to the camera.

## Using it

- **Add receipt:** take or choose a photo, check the fields, write what the expense was for, and tap **Save to OneDrive**. The photo uploads to `Receipts/<year>/`, the ledger updates, and the spreadsheet is rebuilt.
- **Ledger:** filter by month, year or expense type. Tap an entry to see the receipt photo, open it in OneDrive, copy a view-only share link for your bookkeeper, edit, or delete it. Deleted photos go to your OneDrive recycle bin.
- **Several devices:** sign in on each one with the same Microsoft account. Changes are merged safely, so saving from two phones at once won't overwrite either.
- **"The spreadsheet is open in Excel":** Excel locks the file while it's open. Close it and tap **Update spreadsheet**; your receipt is already saved.

## Troubleshooting

- **"The reply URL specified in the request does not match"**: the redirect URI in Azure doesn't exactly match the address in your browser. Copy it again from the setup screen.
- **Nothing happens after signing in**: check the API permission `Files.ReadWrite` was added and saved.
- **Receipt reading says the key was rejected**: re-copy the key from the Anthropic console, without spaces.
- **Wrong date order (day/month swapped)**: correct it in the form before saving; the model guesses from the shop's country and currency.
