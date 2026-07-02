# Silliza — Full Stack Deployment Guide

Your app is now a proper client + server setup:

```
silliza-server/
├── server.js          ← Express backend (API proxy + loan storage)
├── package.json
├── .env.example       ← Copy to .env and fill in your keys
├── railway.toml       ← Railway one-click deploy config
├── render.yaml        ← Render one-click deploy config
├── .gitignore
└── public/            ← Your PWA frontend (index.html, sw.js, etc.)
```

---

## STEP 1 — Set up GitHub (required for all platforms)

```bash
# In the silliza-server/ folder:
git init
git add .
git commit -m "Initial Silliza full-stack app"

# Create a new repo on github.com, then:
git remote add origin https://github.com/YOUR_USERNAME/silliza-server.git
git push -u origin main
```

---

## STEP 2 — Deploy to Railway (Recommended — Free tier)

1. Go to **https://railway.app** and sign up with GitHub
2. Click **New Project → Deploy from GitHub repo**
3. Select your `silliza-server` repo
4. Railway auto-detects Node.js and deploys

**Set environment variables in Railway dashboard:**
- Go to your project → **Variables** tab
- Add these one by one:

| Variable | Value |
|----------|-------|
| `ANTHROPIC_API_KEY` | `sk-ant-your-key-here` |
| `ADMIN_KEY` | `any-strong-password` |
| `MANAGER_WHATSAPP` | `260769309326` |
| `CEO_WHATSAPP` | `260979939322` |

5. Railway gives you a URL like: `https://silliza-server.up.railway.app`
6. Your app is live! The PWA frontend is served at that URL.

---

## STEP 2 (Alternative) — Deploy to Render (also free)

1. Go to **https://render.com** and sign up with GitHub
2. Click **New → Web Service**
3. Connect your `silliza-server` repo
4. Settings:
   - **Build Command:** `npm install`
   - **Start Command:** `node server.js`
5. Add environment variables (same as Railway table above)
6. Click **Create Web Service**
7. You get a URL like: `https://silliza.onrender.com`

> **Note:** Render free tier spins down after 15 min of inactivity.
> Railway free tier stays awake. Railway is recommended.

---

## STEP 3 — Custom Domain (silliza.com or silliza.co.zm)

### On Railway:
1. Go to your project → **Settings → Domains**
2. Click **Add Custom Domain**
3. Enter `silliza.com`
4. Railway shows you a CNAME record to add to your DNS

### In your domain registrar (GoDaddy, Namecheap, ZICTA):
- Add a **CNAME** record:
  - Name: `@` (or `www`)
  - Value: the Railway CNAME they gave you
- Wait 5–30 minutes for DNS to propagate

---

## API Endpoints Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Server status check |
| POST | `/api/chat` | Sila AI agent proxy (rate-limited) |
| POST | `/api/loans` | Save loan application |
| GET | `/api/loans` | List all loans (admin key required) |
| PATCH | `/api/loans/:id` | Update loan status (admin key required) |
| POST | `/api/notify` | Get WhatsApp notification links |

### Example: View all loans (admin)
```bash
curl https://your-app.railway.app/api/loans \
  -H "x-admin-key: your-admin-password"
```

### Example: Approve a loan
```bash
curl -X PATCH https://your-app.railway.app/api/loans/1234567890 \
  -H "x-admin-key: your-admin-password" \
  -H "Content-Type: application/json" \
  -d '{"status": "approved"}'
```

---

## Getting Your Anthropic API Key

1. Go to **https://console.anthropic.com**
2. Sign up / log in
3. Click **API Keys → Create Key**
4. Copy the key — it starts with `sk-ant-`
5. Paste it as `ANTHROPIC_API_KEY` in your Railway/Render variables

> The key is safe on your server. It is never sent to the browser.

---

## Local Testing (Before Deploying)

```bash
# In the silliza-server/ folder:
cp .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY

npm install
node server.js
# Open http://localhost:3000
```

---

## Architecture Summary

```
Browser (PWA)
    │
    ├── GET /          → Loads index.html (your full UI)
    ├── POST /api/chat → Server calls Anthropic API (key is secret)
    ├── POST /api/loans → Loan saved to server + localStorage
    └── POST /api/notify → Returns WhatsApp links
```

The API key never leaves the server. The frontend only talks to `/api/*`.
