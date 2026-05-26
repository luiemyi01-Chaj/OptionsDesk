# OptionsDesk 📊

A private, self-hosted options trading research dashboard. Runs entirely in the browser — no server, no subscription fees.

**Live features:**
- Real-time quotes & options chains (Yahoo Finance)
- Black-Scholes Greeks (Delta, Gamma, Theta, Vega) calculated client-side
- RSI, MACD, Bollinger Bands technical analysis
- Earnings & catalyst calendar
- Trade log with P&L tracking
- Google Sheets sync/backup

---

## Quick Start

### 1 — Fork or clone this repo

```bash
git clone https://github.com/YOUR_USERNAME/optionsdesk.git
cd optionsdesk
```

Open `index.html` directly in your browser for local use. No build step required.

---

### 2 — Deploy to GitHub Pages (free hosting)

1. Push the repo to GitHub
2. Go to **Settings → Pages**
3. Under *Source*, select **GitHub Actions**
4. The included workflow (`.github/workflows/pages.yml`) will auto-deploy on every push to `main`
5. Your app will be live at: `https://YOUR_USERNAME.github.io/optionsdesk/`

> **Privacy note:** GitHub Pages sites are publicly accessible by URL. The app contains no login — keep the URL private or see [Private Deployment](#private-deployment) below.

---

### 3 — Google Sheets Setup (optional — for cloud sync/backup)

Google Sheets is used as a backup database for your trade log and watchlist. This is optional; the app works fully offline using `localStorage`.

#### 3a — Create the spreadsheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet
2. Rename the default sheet tab to **`Trades`**
3. Add two more tabs: **`Watchlist`** and **`Settings`**
4. Note the **Spreadsheet ID** from the URL:
   `https://docs.google.com/spreadsheets/d/`**`SPREADSHEET_ID`**`/edit`

#### 3b — Enable the Google Sheets API

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g., `OptionsDesk`)
3. Navigate to **APIs & Services → Library**
4. Search for **"Google Sheets API"** and click **Enable**

#### 3c — Create an API Key (read-only access)

1. Go to **APIs & Services → Credentials**
2. Click **Create Credentials → API Key**
3. Copy the key — paste it into the app under **Settings → Google Sheets → API Key**
4. (Recommended) Click **Restrict Key** → restrict to *Google Sheets API* only

#### 3d — Create an OAuth 2.0 Client ID (write access)

1. Still in **Credentials**, click **Create Credentials → OAuth 2.0 Client ID**
2. If prompted, configure the **OAuth consent screen** first:
   - User Type: **External**
   - App name: `OptionsDesk`
   - Add your email as a test user
3. Back in Client ID creation:
   - Application type: **Web application**
   - Authorized JavaScript origins: add your GitHub Pages URL
     e.g. `https://YOUR_USERNAME.github.io`
   - Also add `http://localhost` if testing locally
4. Copy the **Client ID** — paste it into **Settings → Google Sheets → OAuth Client ID**

#### 3e — Share the spreadsheet

- In Google Sheets, click **Share**
- Add the email you used for Google Cloud as an **Editor**
- Or set *General access* to **"Anyone with the link"** as Viewer (read is public, writes require OAuth sign-in)

---

### 4 — Configure the app

Open the **Settings** tab in OptionsDesk:

| Field | Where to find it |
|---|---|
| Sheet ID | Google Sheets URL (see 3a) |
| API Key | Google Cloud Credentials (see 3c) |
| OAuth Client ID | Google Cloud Credentials (see 3d) |
| Account Size | Your starting trading capital |
| Max Position Size | % of account per trade (recommended: 2–5%) |
| Max Daily Loss | Hard stop % (recommended: 2–3%) |

Click **Save Settings**, then **Sync to Sheets** to push your local data to Google Sheets.

---

## Watchlist Tickers

Default watchlist: `NVDA`, `AMD`, `TD`, `RY`, `BNS`, `BMO`, `CM`, `GOOGL`, `SPY`

**Canadian banks** are tracked via their **US-listed (NYSE) tickers** so options data is available:

| Bank | US Ticker | Exchange |
|---|---|---|
| TD Bank | `TD` | NYSE |
| Royal Bank | `RY` | NYSE |
| Bank of Nova Scotia | `BNS` | NYSE |
| Bank of Montreal | `BMO` | NYSE |
| CIBC | `CM` | NYSE |

Manage your watchlist in **Settings → Watchlist**.

---

## Market Data

Market data is fetched from the **Yahoo Finance unofficial API** via a CORS proxy. No API key is required.

- Primary proxy: `corsproxy.io`
- Fallback proxy: `allorigins.win`

> These proxies are free and require no registration. If you experience reliability issues, see [Custom CORS Proxy](#custom-cors-proxy) below.

---

## Features Overview

### Dashboard
- Live price cards for all watchlist tickers
- Price change %, 52-week range bar
- Open position count, today's realized P&L, win rate
- Alert panel: flags moves >3% automatically

### Options Chain
- Full chain for any symbol + expiry date
- Filter: Calls / Puts / Both, ±N strikes from ATM, ITM only, High OI (>1000)
- Greeks: Delta, Gamma, Theta, Vega (Black-Scholes)
- IV color-coded: green (low <20%), yellow (mid), red (high >60%)
- One-click "+ Trade" prefills the trade log

### Technical Analysis
- RSI (14), MACD (12,26,9), Bollinger Bands (20, 2σ)
- Signal cards: Bullish / Bearish / Neutral with numeric score
- Charts: Price + Bollinger, RSI with 30/70 bands, MACD histogram

### Earnings Calendar
- Pulls upcoming earnings from Yahoo Finance for watchlist symbols
- Curated links to EarningsWhispers, Barchart Unusual Activity, Market Chameleon

### Trade Log
- Record: symbol, strategy (Long Call/Put, Bull/Bear Spread, Iron Condor, etc.), strike, expiry, contracts, entry price, notes
- Open positions table with DTE color coding
- Close positions with exit price → auto-calculates P&L
- Trade history with return %
- CSV export

### Settings
- Watchlist manager (add/remove tickers)
- Google Sheets configuration
- Risk parameters
- Export/Import full backup as JSON
- Reset all data

---

## Private Deployment

For extra privacy, consider one of these alternatives to public GitHub Pages:

**Option A — Password protect with Cloudflare Access (free)**
1. Add your GitHub Pages domain to Cloudflare
2. Enable **Cloudflare Access** on the path
3. Create a policy: allow only your email

**Option B — Cloudflare Pages (free, private previews)**
1. Deploy to Cloudflare Pages instead of GitHub Pages
2. Preview deployments are private by default

**Option C — Run locally only**
- Just open `index.html` directly in your browser
- All data stays in `localStorage` on your machine
- Use the JSON export for manual backups

---

## Custom CORS Proxy

If the public CORS proxies are unreliable, deploy your own Cloudflare Worker (free tier, 100k req/day):

```javascript
// workers/cors-proxy.js
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const target = decodeURIComponent(url.searchParams.get('url'));
    const res = await fetch(target, {
      headers: { 'User-Agent': 'Mozilla/5.0' }
    });
    return new Response(res.body, {
      headers: {
        'Content-Type': res.headers.get('content-type'),
        'Access-Control-Allow-Origin': '*'
      }
    });
  }
}
```

Deploy via `wrangler deploy`, then update `CFG.PROXY` in `index.html` line ~20.

---

## Tech Stack

| Component | Technology |
|---|---|
| UI framework | Vanilla JS (no dependencies) |
| Charts | Chart.js 4 (CDN) |
| Fonts | JetBrains Mono, DM Sans (Google Fonts) |
| Market data | Yahoo Finance (unofficial) |
| Storage | localStorage + Google Sheets API v4 |
| Hosting | GitHub Pages |
| CI/CD | GitHub Actions |

---

## Disclaimer

This application is for personal research and record-keeping only. It does not constitute financial advice. Options trading involves significant risk. Past performance does not guarantee future results. Always do your own due diligence.
