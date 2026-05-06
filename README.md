# web.juriekriel.info — Netlify dashboard

A personal admin page for Jurie's Netlify sites. Lists all sites, shows last-deploy
status and time, and lets you drop an `.html` or `.zip` file onto any site to deploy
it instantly.

Mirrors the `sn.juriekriel.info` setup pattern: Cloudflare DNS → Netlify hosting.

## Architecture

Single static HTML file in `public/index.html`. No backend, no build step, no
`npm install`. The page calls `api.netlify.com` directly from the browser using
your **Netlify Personal Access Token**, which is stored only in this browser's
`localStorage`. The token never leaves your machine.

Why no backend: Netlify's synchronous Functions cap requests at ~6 MB, which would
silently break uploads of larger files (the NXT Move standalone is 12 MB). Going
direct browser → Netlify removes that ceiling.

## Setup (one-time)

1. **Create a GitHub repo** at `juriekriel/web-dashboard` (Public is fine — no
   secrets in this code). Push these files to it on `main`.

2. **Create a Netlify site** connected to the new repo. Build command: leave empty.
   Publish directory: `public`. The `netlify.toml` already specifies these.

3. **Generate a Netlify Personal Access Token**:
   - Go to https://app.netlify.com/user/applications#personal-access-tokens
   - Click **New access token**, name it "Dashboard", copy the value (`nfp_…`).

4. **Custom domain**:
   - In the new Netlify site → **Domain management** → **Add a domain alias**
     → `web.juriekriel.info`. Netlify will tell you the CNAME target
     (something like `your-site-name.netlify.app`).
   - In Cloudflare DNS → add a **CNAME** record:
     - Name: `web`
     - Target: the Netlify edge hostname from the previous step
     - Proxy status: Proxied (orange cloud) — same as `sn`.
   - Wait ~2 minutes for DNS to propagate, then Netlify will issue the SSL cert.

5. **First visit**: open `https://web.juriekriel.info`, paste the PAT, done.

## Daily use

Open `https://web.juriekriel.info`. You'll see all your Netlify sites with a
green/red/yellow status dot, the URL, and how long ago the last deploy was.

To deploy: drag an `.html` or `.zip` onto any site row. The dashboard:
- For an `.html` file: deploys it as the new `/index.html` for that site.
- For a `.zip` file: forwards the zip directly to Netlify (full multi-file deploy).

The status auto-refreshes every 30 seconds.

## Security notes

- The dashboard URL is unindexed (`X-Robots-Tag: noindex, nofollow, noarchive`)
  but technically reachable by anyone who knows it. The PAT in `localStorage`
  protects against use, not against people knowing the URL exists.
- If you ever lose your Mac or suspect compromise: revoke the PAT at
  [netlify.com/user/applications](https://app.netlify.com/user/applications#personal-access-tokens).
  The dashboard will start failing within seconds.
- The token is account-wide — anything you can do at `app.netlify.com` (delete
  sites, change DNS, etc.) the dashboard could in theory do too. The current UI
  only exposes list + deploy, but treat the token as you would your Netlify
  password.

## Local development

Open `public/index.html` directly in a browser (it works as a `file://` URL or
through any static server). The PAT prompt comes up first time; everything else
works identically to production.
