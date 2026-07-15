# JEE Battle Royale Tracker — Offline App Bundle

## What changed for offline use

- **Auto-save to device storage**: the app no longer depends on Claude's `window.storage`
  (which only exists inside claude.ai). It now uses the browser's built-in `localStorage`
  automatically whenever that Claude-specific API isn't present — which is the case once
  this is opened as its own app. Every checklist tick, test, and daily log saves instantly,
  with zero setup, and works with no internet connection at all.
- **Backup/Restore stays as a bonus**, not a requirement — useful only if you switch phones
  or want an extra copy. Since data now lives on the device itself, you won't lose anything
  by just closing the app.
- **Offline installable (PWA)**: `manifest.json` + `sw.js` were added so the app can be
  "installed" (Add to Home Screen) and fully cached for offline use, which is also the
  standard first step toward turning it into a real APK (see below).
- The shared **Leaderboard** tab relied on Claude's shared cloud storage across users —
  that piece has no equivalent once fully offline/standalone, so it will just show your
  own device's data (or stay empty) instead of syncing with other people.

## Files in this bundle
```
index.html      → the app itself
manifest.json   → app name/icon/theme metadata (PWA)
sw.js           → offline caching (service worker)
icon-192.png    → app icon (small)
icon-512.png    → app icon (large)
```
Keep all five files together, in the same folder, when hosting or packaging.

## Turning this into an actual installable APK

An APK is an Android package — Claude can't compile and sign one directly inside this chat
(that needs the Android SDK/build tools, which aren't available here). But since this is
already a working offline-capable PWA, there are two realistic, low-effort paths:

### Option A — PWABuilder (easiest, no coding, browser-only)
1. Host these 5 files somewhere reachable by a URL — even a free static host works
   (GitHub Pages, Netlify, Vercel, Cloudflare Pages — drag-and-drop the folder in).
2. Go to **https://www.pwabuilder.com**, paste your hosted URL, and let it scan the site.
3. Click **Package for Stores → Android**. It will generate a signed (or ready-to-sign) APK/AAB
   using your `manifest.json` and icons automatically.
4. Download the APK and install it on an Android phone (enable "install from unknown sources"
   if prompted), or upload the AAB to the Play Store.

### Option B — Bubblewrap CLI (if you want it fully local/offline)
Requires Node.js, and Bubblewrap will download the Android SDK + JDK on first run (needs
internet once, for setup only):
```bash
npm install -g @bubblewrap/cli
bubblewrap init --manifest https://your-hosted-url/manifest.json
bubblewrap build
```
This produces a signed APK you can sideload directly — no app store needed.

Either way, once installed as an APK, the app runs fully offline and saves data straight to
the phone's local storage automatically, exactly as it does in the browser version — there's
nothing extra to configure.
