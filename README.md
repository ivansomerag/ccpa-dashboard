# CCPA Dashboard — Static Vercel Deployment

A standalone, zero-infrastructure dashboard for CCPA compliance data. No server required — data is pushed from the local Windows PC via `github_uploader.py` and served as static JSON from this repo.

## How It Works

```
Local PC (CCPA_Toolkit_Dev)
  └── app.py runs validator
        └── dashboard_cache.json updated
              └── github_uploader.py pushes to this repo
                    └── Vercel serves index.html
                          └── index.html fetches dashboard_data.json from GitHub raw URL
```

## One-Time Setup

### 1. Create the GitHub repo

Create a **public** GitHub repository named `ccpa-dashboard` (or any name you prefer).

### 2. Push this folder to GitHub

```bash
cd ccpa-dashboard/
git init
git add .
git commit -m "Initial dashboard deploy"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/ccpa-dashboard.git
git push -u origin main
```

### 3. Update the data URL in index.html

Open `index.html` and replace the `DATA_URL` constant near the top:

```js
const DATA_URL = "https://raw.githubusercontent.com/YOUR_USERNAME/ccpa-dashboard/main/dashboard_data.json";
```

Replace `YOUR_USERNAME` with your GitHub username.

### 4. Configure github_uploader.py

In `CCPA_Toolkit_Dev/github_uploader.py`, fill in the config block at the top:

```python
GITHUB_TOKEN  = "ghp_your_actual_token"   # GitHub Personal Access Token (repo scope)
GITHUB_OWNER  = "your-github-username"
GITHUB_REPO   = "ccpa-dashboard"
GITHUB_FILE   = "dashboard_data.json"
```

To create a token: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → New token → select `repo` scope.

### 5. Deploy on Vercel

1. Go to [vercel.com](https://vercel.com) and log in
2. Click **Add New → Project**
3. Import your `ccpa-dashboard` GitHub repository
4. Leave all settings at defaults (Vercel auto-detects static HTML)
5. Click **Deploy**

Your dashboard will be live at `https://ccpa-dashboard-xxx.vercel.app` (or your custom domain).

### 6. Commit the updated index.html

After updating `DATA_URL`, push the change:

```bash
git add index.html
git commit -m "Set data URL to GitHub raw endpoint"
git push
```

Vercel will auto-redeploy within ~30 seconds.

## How Data Updates

Every time the CCPA validator runs successfully in the local toolkit:

1. `app.py` calls `github_uploader.py` as a background subprocess
2. `github_uploader.py` reads `dashboard_cache.json` and pushes it to this repo as `dashboard_data.json`
3. The dashboard auto-refreshes every **5 minutes**, picking up the new data

You can also force a refresh by clicking the **Refresh** button in the dashboard header.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Complete standalone dashboard (React 18 + Recharts via CDN) |
| `dashboard_data.json` | Live data file — overwritten by `github_uploader.py` on each run |
| `README.md` | This file |

## Tech Stack

- **React 18.2** via CDN (no npm, no Node.js, no build step)
- **Recharts 2.12.7** via CDN for charts
- **Google Fonts** (DM Sans + JetBrains Mono) via CDN
- **GitHub Contents API** for data delivery
- **Vercel** for static hosting (free tier)
