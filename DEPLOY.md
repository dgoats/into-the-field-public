# Into the Field — Deployment Guide

## Why a proxy?
Browsers block direct connections to databases (CORS policy). The Vercel proxy
is a tiny middleman: your pages call it, it calls Neon, and returns the result.
It's free, takes ~5 minutes to set up, and keeps your DB credentials off the browser.

---

## Step 1 — Create a free Vercel account
Go to https://vercel.com and sign up (GitHub login is easiest).

---

## Step 2 — Deploy the proxy

### Option A — Vercel CLI (fastest)
```bash
npm i -g vercel
cd vercel-proxy
vercel deploy --prod
```

### Option B — Drag & drop (no terminal needed)
1. Go to https://vercel.com/new
2. Choose "Deploy without a Git repository"  
3. Drag the entire `vercel-proxy` folder onto the upload area
4. Click Deploy

After deployment Vercel gives you a URL like:
  https://into-the-field-proxy.vercel.app

---

## Step 3 — Add your Neon connection string to Vercel

1. In the Vercel dashboard, open your project
2. Go to **Settings → Environment Variables**
3. Add a new variable:
   - **Name:** `NEON_CONNECTION_STRING`
   - **Value:** your full Neon connection string, e.g.  
     `postgresql://alex:password@ep-cool-123.us-east-2.aws.neon.tech/neondb?sslmode=require`
   - Select all three environments (Production, Preview, Development)
4. Click Save
5. Go to **Deployments → Redeploy** (so the env var takes effect)

Your Neon connection string is in:
  Neon Console → your project → Connect button

---

## Step 4 — Update the HTML files

Open **admin.html** and find this line near the top of the `<script>`:
```javascript
const PROXY_URL = 'YOUR_VERCEL_URL/api/neon';
```
Replace with your actual URL:
```javascript
const PROXY_URL = 'https://into-the-field-proxy.vercel.app/api/neon';
```

Do the same in **public.html**.

---

## Step 5 — Set up the database tables

1. Open admin.html in your browser
2. Log in (default password: `inthefield2025`)
3. Click **"Setup tables"** — this creates the Neon table with one click
4. Click **"Test connection"** — should say "Connected successfully"

---

## Step 6 — Upload the artwork

1. In the admin sidebar, click the artwork upload area
2. Choose Frida Foberg's image (square JPG/PNG works best)
3. Click **"Save image to database"**

The image is stored in Neon and the public page will load it automatically.

---

## Step 7 — Embed in WordPress

Upload public.html to your server (e.g. via FTP to /wp-content/fundraiser/) then:

```html
<iframe 
  src="https://yoursite.com/wp-content/fundraiser/public.html"
  width="100%" 
  height="900"
  frameborder="0"
  style="border:none;"
></iframe>
```

Keep admin.html password-protected and never embed it publicly.

---

## How donations trigger a reveal

**Manual:** Click "Record donation" on the admin page.

**Automatic (via webhook/Zapier):** Connect your donation platform to run this
SQL each time a donation comes in — call your proxy endpoint:

```
POST https://into-the-field-proxy.vercel.app/api/neon
Content-Type: application/json

{ "query": "UPDATE fundraiser_state SET value = (value::int + 1)::text, updated_at = NOW() WHERE key = 'donation_count'" }
```

The public page picks it up within 6 seconds automatically.
