# Study Notes — PWA package

This folder is a ready-to-host Progressive Web App: `index.html` (your app),
`manifest.json`, `sw.js` (offline support), plus `/icons` and `/screenshots`.

All your data still lives in the browser's local storage on whatever device
opens the app — there is no server component and nothing to configure there.

## 1. Host it somewhere with HTTPS

PWABuilder (and both app stores) need a real HTTPS URL to inspect — you can't
package straight from a local file. Easiest free options, in order of
simplicity:

- **Netlify Drop** — netlify.com/drop — drag this whole folder into the
  browser tab, get a URL in seconds. No account required to try it.
- **Cloudflare Pages** or **Vercel** — free tier, connect a GitHub repo or
  drag-and-drop.
- **GitHub Pages** — push this folder to a repo, enable Pages in Settings.

Whichever you pick, once it's live, open the URL and confirm the app loads
normally.

## 2. Check it's a valid, installable PWA

Go to https://www.pwabuilder.com, paste your URL, and click Start. It scores
your manifest, service worker, and icons and flags anything missing. This
package is already filled in (icons, screenshots, offline support, shortcuts),
so you should see a clean report.

## 3. Package for Android

**Option A — pwabuilder.com (no install, easiest):**
1. From your PWABuilder report page, click **Package for Store → Android**.
2. Leave the defaults (they're read from manifest.json) and click **Generate**.
3. Download the zip — it contains a signed `.aab` (for Google Play) and a
   `.apk` (for sideloading/testing) plus your signing key — keep that key safe,
   you'll need the same one for every future update.
4. Create a Google Play Developer account (one-time $25 fee) at
   play.google.com/console, create a new app, and upload the `.aab`.

**Option B — Bubblewrap CLI (JavaScript/Node, runs on your own machine):**
```bash
npm i -g @bubblewrap/cli
bubblewrap init --manifest https://YOUR-DOMAIN/manifest.json
# answer the prompts (it reads most of this from manifest.json already);
# first run offers to install the JDK + Android SDK for you — say yes
bubblewrap build
```
This produces `app-release-signed.apk` (install directly on a device or
emulator to test) and `app-release-bundle.aab` (upload this one to Play
Console). Run `bubblewrap update` any time you change the manifest.

**A note on timing:** Google Play now requires apps to target Android 16
(API level 36). Some PWABuilder/Bubblewrap templates still default to 35 —
if your Play Console upload is rejected for target SDK, open the generated
Android project (`android/app/build.gradle`) and bump `targetSdkVersion` to
36, then rebuild.

## 4. Package for Windows

There isn't a well-maintained Node CLI for this step (the old `pwabuilder`
npm package predates the current MSIX/WebView2 packaging and isn't
recommended) — pwabuilder.com's own packaging service is the standard route,
so this is the one step done through the browser rather than the terminal:

1. From your PWABuilder report page, click **Package for Store → Windows**.
2. If you plan to submit to the Microsoft Store, first reserve an app name at
   partner.microsoft.com/dashboard (Store apps → New product), then copy the
   **Package Identity** values (Package/Publisher ID, Publisher display name)
   it gives you into the Windows packaging form. If you only want to sideload
   it yourself, you can leave the defaults.
3. Click **Generate** and download the zip.
4. To test locally: unzip it, right-click `install.ps1` → **Run with
   PowerShell** (if scripts are blocked, run `Set-ExecutionPolicy Bypass` in
   an admin PowerShell first). It installs and launches the app like a
   normal Windows program.
5. To publish: upload the `.msixbundle`/`.appxbundle` from the same zip to
   your Partner Center product page.

## Updating the app later

Edit `index.html`, re-upload it to your host, and bump `CACHE_NAME` at the
top of `sw.js` (e.g. `study-notes-cache-v2`) so installed copies fetch the
new version instead of serving the old cached one. The Android/Windows
packages just point at your hosted URL, so you only need to repackage them
if you change the manifest (name, icons, colors) — everyday app updates
don't require a new store submission at all.
