# ⚡ Shopify 6K Payment Gateway Checker

A high-performance Shopify checkout verification service ready for **Railway** deployment with automatic domain selection, optional proxy routing, and multi-threaded processing.

---

## 🚀 Features

- **Ready for Railway**: Fully containerized with `Dockerfile`, `railway.json`, `Procfile`, and healthcheck probes.
- **Automatic Domain Generation**:
  - If no `site` or `domain` parameter is passed in the request URL, the service automatically selects and tests against an active Shopify store from `shopify_sites.txt` or configured defaults.
- **Optional Proxy Support**:
  - `proxy` is 100% optional in the URL. If omitted, checks execute directly without requiring proxies.
  - Supports `ip:port`, `ip:port:user:pass`, `http://...`, and `socks5://...`.
- **Dual Endpoints & Aliases**:
  - `GET/POST /shopify`
  - `GET/POST /shopii`
  - `GET/POST /check`
  - `GET /health` (Railway health check)
  - `GET /` (Interactive Web Dashboard & Live API Playground)
- **High Concurrency**: Powered by [Waitress](https://docs.pylonsproject.org/projects/waitress/) WSGI server configured for 200 worker threads and 1,000 concurrent channels.

---

## 🛠 Railway Deployment Guide

### Method 1: Using the Automated Script (Easiest)

If you have the [Railway CLI](https://docs.railway.app/guides/cli) installed:

```bash
# 1. Install Railway CLI (if not already installed)
npm i -g @railway/cli

# 2. Login to Railway
railway login

# 3. Run the automated deployment and domain generator script
python railway_deploy.py
```

The script will:
1. Initialize the Railway project.
2. Deploy the container.
3. Automatically execute `railway domain` to provision your free `*.up.railway.app` public domain.

---

### Method 2: Deploying via Railway Dashboard / GitHub

1. **Push your code to GitHub**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit for Railway"
   git remote add origin https://github.com/your-username/shopify-checker.git
   git push -u origin main
   ```
2. **Go to [railway.app](https://railway.app)**:
   - Click **+ New Project** -> **Deploy from GitHub repo**.
   - Select your repository.
3. **Generate Public Domain**:
   - In Railway dashboard, click on your deployed service.
   - Go to **Settings** -> **Networking** (or **Public Networking**).
   - Click **Generate Domain**. Railway will instantly assign a public URL like:
     ```
     https://shopify-checker-production.up.railway.app
     ```

---

## 🌐 API Reference

### Base URL:
`https://<your-railway-domain>.up.railway.app`

### 1. Check Card Endpoint
`GET` or `POST` `/shopify` (or `/shopii`, `/check`)

#### Query Parameters / JSON Body:

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `cc` | String | **Yes** | Card details in format: `NUM\|MM\|YYYY\|CVV` or `NUM\|MM\|YY\|CVV` |
| `site` / `domain` | String | *Optional* | Shopify store URL. If omitted, automatically selects an active store. |
| `proxy` | String | *Optional* | Proxy in `ip:port` or `ip:port:user:pass`. If omitted, requests are direct. |

---

### Request Examples

#### 1. Minimal Request (Auto-Domain + No Proxy):
```bash
curl "https://your-service.up.railway.app/shopify?cc=4532015112830366|05|2027|000"
```

#### 2. With Custom Shopify Domain (Optional Proxy):
```bash
curl "https://your-service.up.railway.app/shopify?cc=4532015112830366|05|2027|000&site=https://shop.gymshark.com"
```

#### 3. With Proxy & Custom Domain:
```bash
curl "https://your-service.up.railway.app/shopify?cc=4532015112830366|05|2027|000&site=https://shop.gymshark.com&proxy=user:pass@1.2.3.4:8080"
```

#### 4. JSON POST Request:
```bash
curl -X POST "https://your-service.up.railway.app/shopify" \
  -H "Content-Type: application/json" \
  -d '{
    "cc": "4532015112830366|05|2027|000",
    "proxy": "1.2.3.4:8080"
  }'
```

---

### Sample Response:
```json
{
  "Cc": "4532015112830366|05|2027|000",
  "Gateway": "Shopify Payments",
  "Price": "25.00",
  "Currency": "USD",
  "Site": "https://shop.gymshark.com",
  "Domain": "https://shop.gymshark.com",
  "AutoDomain": true,
  "Proxy": "Direct",
  "Response": "CARD_DECLINED",
  "Status": "false"
}
```

---

## ⚙️ Environment Variables (Optional)

Configure these in the Railway Dashboard under **Variables**:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `PORT` | `8003` (Railway injects automatically) | Port for the waitress server |
| `DEFAULT_SITE` | *Empty* | Fixed fallback Shopify store URL |
| `DEFAULT_SHOPIFY_DOMAINS` | Built-in list | Comma-separated list of fallback Shopify store URLs |
| `HUENCY_THREADS` | `200` | Waitress worker thread count |
| `HUENCY_CHANNELS` | `1000` | Waitress concurrent connection limit |

---

## 📂 Project Structure

```
├── Dockerfile             # Production container definition for Railway
├── railway.json           # Railway deployment & health check configuration
├── railway.toml           # Alternative Railway configuration
├── Procfile               # Standard PaaS process definition
├── requirements.txt       # Python dependencies (Flask, waitress, curl_cffi, requests)
├── .dockerignore          # Build context exclusion rules
├── railway_deploy.py      # CLI deployment & automated domain generation script
├── shopify_sites.txt      # List of default Shopify stores for auto-selection
├── main.py                # Core application & API service
└── README.md              # Documentation
```
