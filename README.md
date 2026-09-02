# Kestrel

A pocket AI agent: static frontend (`index.html`) + a small serverless
backend (`api/chat.js`) that holds your Anthropic API key so it's never
exposed in the browser.

## Deploy on Vercel (free tier is enough for personal use)

1. **Get an API key** at https://console.anthropic.com — API Keys →
   Create Key. Copy it somewhere safe.

2. **Push this folder to GitHub.**
   ```bash
   git init
   git add .
   git commit -m "Kestrel"
   git branch -M main
   git remote add origin https://github.com/<you>/kestrel.git
   git push -u origin main
   ```

3. **Import into Vercel.**
   - Go to https://vercel.com → New Project → Import your `kestrel` repo.
   - Framework preset: "Other" (no build step needed).
   - Don't deploy yet — first add the environment variable below.

4. **Add your API key as an environment variable.**
   - In the Vercel project → Settings → Environment Variables
   - Name: `ANTHROPIC_API_KEY`
   - Value: the key from step 1
   - Apply to: Production, Preview, and Development
   - Save, then deploy (or redeploy).

5. **You'll get a live URL** like `https://kestrel-yourname.vercel.app`.
   Open it on your phone — it already works like a home-screen app if you
   "Add to Home Screen" from the browser share menu (this alone gives you
   a full-screen icon-launched app with no Play Store needed).

## Alternative hosts

Any host that supports a static site + one serverless function works the
same way: Netlify (Functions), Cloudflare Pages (Pages Functions), or
Render. The only thing that changes is where you set the
`ANTHROPIC_API_KEY` environment variable and the exact function file
location/format.

## Monetizing with ads

The frontend already has one ad slot wired up (a slim banner above the
message box) using Google AdSense. To turn it on:

1. **Apply at** https://adsense.google.com **with your live site URL**
   (you need the real deployed URL from step 5 above, not localhost).
   Review can take anywhere from a day to a couple weeks.
2. Once approved, get your **publisher ID** (Account → Settings, looks like
   `pub-1234567890123456`) and replace the three `ca-pub-XXXXXXXXXXXXXXXX`
   placeholders in `index.html` with `ca-pub-<your-id>`.
3. Create a **Display ad unit** (Ads → By ad unit → Display ads →
   Responsive) and put its **slot ID** into `data-ad-slot` in `index.html`.
4. Replace `pub-XXXXXXXXXXXXXXXX` in `ads.txt` with your publisher ID and
   redeploy. AdSense won't serve reliably without this file matching.
5. If you expect visitors in the EU/UK, AdSense requires a Google-certified
   consent banner before ads can show there (Privacy & messaging →
   Consent management in your AdSense dashboard sets this up for you —
   no extra code needed on your end).

Until step 2 is done, the ad script fails silently and the slot just stays
empty — safe to deploy before your account is approved.

**Before submitting to Play Store with ads enabled:** your store listing
must disclose that the app shows ads and what data it collects (Play
Console → App content → Ads / Data safety section), and you need a
privacy policy URL that mentions this — required either way for
publishing, but doubly so once ads and their tracking are involved.

## Play Store packaging (after it's hosted)

The app now has everything a Trusted Web Activity (TWA) needs: a web app
manifest (`manifest.json`), icons (`/icons`), and a service worker
(`sw.js`) so it passes installability checks. A TWA is a thin native
Android shell that launches your live site full-screen with no browser
address bar — that's what actually gets uploaded to Play Store.

1. **Install Bubblewrap** (needs Node.js and a JDK installed on your
   computer):
   ```bash
   npm i -g @bubblewrap/cli
   ```

2. **Generate the Android project** from your live URL:
   ```bash
   bubblewrap init --manifest https://<your-live-url>/manifest.json
   ```
   It'll ask a few questions (package name like `com.yourname.kestrel`,
   app name, etc.) — defaults are usually fine. This step also generates
   a signing key; keep the `.keystore` file and its password safe, you
   need the same one for every future update.

3. **Build it:**
   ```bash
   bubblewrap build
   ```
   This produces a signed `.aab` file — the format Play Store requires.

4. **Verify domain ownership** (required or the TWA shows a browser
   address bar instead of running full-screen). Bubblewrap generates an
   `assetlinks.json` file during `init` — copy it into a `.well-known`
   folder in this project so it's served at
   `https://<your-live-url>/.well-known/assetlinks.json`, then push and
   redeploy before uploading to Play Store.

5. **Create your Play Console listing.** You'll need: the $25 one-time
   developer account fee, the `.aab` from step 3, an app icon (use
   `icons/icon-512.png`), a few screenshots of the app in use, a short
   and long description, a privacy policy URL, and — since ads are
   enabled — the Ads and Data Safety disclosures under App content.

## Local development

```bash
npm i -g vercel
vercel dev
```

This runs both the static site and the `/api/chat` function locally at
`http://localhost:3000`, reading `ANTHROPIC_API_KEY` from a local `.env`
file (create one — it's gitignored).
