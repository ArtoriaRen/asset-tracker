# Asset Tracker — Setup Guide

This is a small web app (a "PWA") that syncs live with your Google Sheet. It reads
whatever columns your sheet has, shows a total and a breakdown by category, and lets
you add / edit / delete rows right from your phone — the changes land in the actual
spreadsheet.

Because it talks to Google Sheets on your behalf, Google requires it to be registered
as a real (if tiny) web app before it can sign you in. That's most of the setup below.
Total time: about 10 minutes, one-time.

## What you have

```
asset-tracker-pwa/
  index.html      <- the whole app (edit one line here: your Client ID)
  manifest.json   <- makes it installable as an app icon
  sw.js           <- lets it open instantly / show cached data offline
  icons/          <- app icon
  SETUP.md        <- this file
```

## Step 1 — Put the files somewhere with a URL

Google's sign-in requires the app to be served over `https://` (or `localhost` while
testing) — it won't work opened as a local `file://` page on your phone. The easiest
free option is **GitHub Pages**:

1. Create a free GitHub account if you don't have one: https://github.com/signup
2. Create a new repository (e.g. `asset-tracker`) — public or private both work.
3. Upload all the files in this folder to that repository (drag-and-drop them on the
   repo's page via "Add file → Upload files", keeping the `icons/` folder structure).
4. Go to the repo's **Settings → Pages**, set "Source" to the `main` branch, root
   folder, and save.
5. GitHub will give you a URL like `https://yourusername.github.io/asset-tracker/`.
   That's your app's address — open it once now, you should see a "Setup needed"
   banner (expected, since you haven't added a Client ID yet).

(Alternative: if you don't want a GitHub account, https://app.netlify.com/drop lets
you drag the folder in and get a URL instantly — no login required for a one-off
static site, though you'll want an account to update it later.)

## Step 2 — Create a Google Cloud project

1. Go to https://console.cloud.google.com/ and sign in with the same Google account
   that owns the spreadsheet.
2. Click the project dropdown at the top → **New Project**. Name it anything (e.g.
   "Asset Tracker") → Create.
3. Make sure that new project is selected in the dropdown.

## Step 3 — Enable the Google Sheets API

1. In the search bar at the top, type **Google Sheets API** and open it.
2. Click **Enable**.

## Step 4 — Configure the OAuth consent screen

1. In the left menu: **APIs & Services → OAuth consent screen**.
2. User type: **External** → Create.
3. Fill in the required fields (app name "Asset Tracker", your email for the two
   contact fields). Save and continue through the "Scopes" step without adding
   anything (default is fine) → Save and continue.
4. On the **Test users** step, click **Add users** and add your own Google email
   address. Save and continue.
5. You can leave the app in "Testing" mode — it's just for you, so it never needs to
   go through Google's verification review.

## Step 5 — Create the OAuth Client ID

1. Left menu: **APIs & Services → Credentials**.
2. **Create Credentials → OAuth client ID**.
3. Application type: **Web application**. Name it anything.
4. Under **Authorized JavaScript origins**, click **Add URI** and enter the URL from
   Step 1 *without* a trailing path — just the origin, e.g.:
   `https://yourusername.github.io`
   (If you're testing locally first, also add `http://localhost:8080` or whatever
   port you use.)
5. Click **Create**. Copy the **Client ID** shown (ends in
   `.apps.googleusercontent.com`).

## Step 6 — Drop the Client ID into the app

Open `index.html`, find this block near the bottom (search for `CONFIG`):

```js
const CONFIG = {
  GOOGLE_CLIENT_ID: "YOUR_GOOGLE_OAUTH_CLIENT_ID.apps.googleusercontent.com",
  DEFAULT_SPREADSHEET_ID: "1M6QumdJ0TzUD_ogxrGEHJmNYAtzcAJ97m2pXCqP3kMk",
  DEFAULT_GID: "0",
};
```

Replace `GOOGLE_CLIENT_ID` with the Client ID you just copied. The spreadsheet ID is
already pre-filled from the sheet you linked (`.../d/`**`1M6Qum...`**`/edit#gid=0`) —
change it (or the `gid`, which is the tab number after `#gid=` in the URL) if you
ever want to point the app at a different sheet or tab. You can also change these
later from inside the app itself, under **Sheet settings**, without editing code.

Re-upload the changed `index.html` to GitHub Pages (or re-drag the folder to
Netlify) to publish the update.

## Step 7 — Open it and sign in

1. Visit your app's URL again (from Step 1).
2. Tap **Sign in with Google**. You'll likely see a warning that "Google hasn't
   verified this app" — that's expected for a personal project in testing mode.
   Click **Advanced → Go to Asset Tracker (unsafe)** to continue. It's only unsafe
   in the sense that Google hasn't manually reviewed it; you wrote it, or had it
   written for you, so you know what it does.
3. Approve access to Google Sheets. You should see your assets load in.

## Step 8 — Install it as an app on Android

1. Open the app's URL in **Chrome** on your Android phone.
2. Tap the **⋮** menu → **Add to Home screen** (or **Install app**, depending on
   your Chrome version).
3. It'll appear on your home screen with its own icon and open full-screen, like a
   native app.

## How it works with your spreadsheet

- The **first row** of your sheet's tab is treated as column headers — the app reads
  whatever columns you already have, no fixed schema required.
- It looks for a column named something like *Value*, *Amount*, *Balance*, *Worth*,
  or *Price* to total up and show as your net worth at the top.
- It looks for a column named *Category* (or *Type* / *Class*) to draw the
  by-category breakdown. Both are optional — the app still works without them, just
  without those two summaries.
- Adding, editing, or deleting an asset in the app writes directly to that row in
  your Google Sheet via the Sheets API — open the sheet in a browser and you'll see
  it update.
- You stay signed in for about an hour at a time; the app tries to refresh access
  quietly in the background, and otherwise just asks you to tap sign-in again.

## Troubleshooting

- **"Setup needed" banner won't go away** — double check you saved `index.html`
  with the real Client ID (no extra spaces) and re-uploaded it.
- **"redirect_uri_mismatch" or sign-in popup fails** — the Authorized JavaScript
  origin in Step 5 has to match your app's URL *exactly* (scheme + domain, no
  trailing slash, no path).
- **Sheet loads empty** — make sure the Google account you signed in with is the
  same one that owns (or has edit access to) the spreadsheet.
- **Want to change which sheet it points to** — use the "Sheet settings" screen in
  the app (from the sign-in screen, or add a settings entry point later) rather than
  editing code each time.
