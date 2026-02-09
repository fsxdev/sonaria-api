# Creatures of Sonaria API - Complete Setup Guide

**This worker completely bypasses the wiki and serves data as YOUR own API.**

## 🎯 How It Works

1. **No Wiki CORS Issues** - Worker fetches data server-side  
2. **Appears as Your API** - No wiki references in responses  
3. **Auto-Updates** - CRON job refreshes data every hour  
4. **Fast** - Data is cached for 1 hour  
5. **No Subrequest Limits** - Fetches one JSON file instead of hundreds of wiki pages  

## 📦 Two Setup Methods

### Method 1: External JSON (Recommended - Auto-Updates)

**Step 1:** Create a GitHub repository for your data

```bash
# Option A: Using GitHub CLI
gh repo create sonaria-data --public
cd sonaria-data

# Create data.json with your scraped data
# (Use the Python scraper or paste manually)
echo '{"metadata":{},"creatures":[...],"plushies":[...]}' > data.json

git add data.json
git commit -m "Initial data"
git push
```

**Step 2:** Get the raw JSON URL

```
https://raw.githubusercontent.com/YOUR_USERNAME/sonaria-data/main/data.json
```

**Step 3:** Update worker.js

```javascript
const DATA_SOURCE_URL = "https://raw.githubusercontent.com/YOUR_USERNAME/sonaria-data/main/data.json";
```

### Method 2: Embedded Data (Simpler - Manual Updates)

**Step 1:** Open worker.js

**Step 2:** Replace `EMBEDDED_DATA` with your complete data:

```javascript
let EMBEDDED_DATA = {
  "metadata": { /* your metadata */ },
  "creatures": [ /* all your creatures */ ],
  "plushies": [ /* all your plushies */ ],
  // etc...
};
```

**Step 3:** Ensure DATA_SOURCE_URL is null:

```javascript
const DATA_SOURCE_URL = null;
```

## 🚀 Deploy

```bash
wrangler deploy
```

## ✅ Test It

```bash
# Should return all your creatures
curl https://your-worker.workers.dev/api/creatures

# Should return specific creature
curl https://your-worker.workers.dev/api/creatures/Aereis

# API documentation
curl https://your-worker.workers.dev/api
```

## 📊 Available Endpoints

```
GET /api/creatures          - All creatures
GET /api/creatures/:name    - Specific creature  
GET /api/plushies          - All plushies
GET /api/mutations         - All mutations
GET /api/palettes          - All color palettes
GET /api/tokens            - All tokens
GET /api/all               - Everything at once
GET /api/stats             - API metadata
GET /api/refresh           - Force cache refresh (testing mode only)
```

## 🔄 Updating Your Data

### If using GitHub (External JSON):

```bash
# 1. Update your data file
python scraper.py  # Or edit manually

# 2. Push to GitHub
cd sonaria-data
cp /path/to/new-data.json data.json
git add data.json
git commit -m "Update data"
git push

# 3. Worker auto-refreshes within 1 hour
# Or force immediate refresh:
curl https://your-worker.workers.dev/api/refresh
```

### If using Embedded Data:

```bash
# 1. Update EMBEDDED_DATA in worker.js
# 2. Redeploy
wrangler deploy
```

## ⚙️ Auto-Update Configuration

The worker refreshes data every hour via CRON:

```toml
# wrangler.toml
[triggers]
crons = ["0 * * * *"]  # Every hour
```

**Change update frequency:**
```toml
crons = ["0 */6 * * *"]   # Every 6 hours
crons = ["0 0 * * *"]     # Daily at midnight
crons = ["0 2 * * *"]     # Daily at 2 AM
```

## 🔒 Security Configuration

### Development Mode (Current):
```javascript
const TESTING_MODE = true;
```
- Allows all CORS origins (`*`)
- Shows detailed error messages
- Enables `/api/refresh` endpoint
- No domain restrictions

### Production Mode:
```javascript
const TESTING_MODE = false;

const ALLOWED_ORIGINS = [
  'https://yourwebsite.com',
  'https://www.yourwebsite.com',
  'https://yourgame.com'
];
```
- Domain whitelist enforced
- Rate limiting active (100 req/min per IP)
- No debug information
- `/api/refresh` disabled

## 🎯 Why This Approach?

### ✅ Solves Your Problems:
- **No "Too many subrequests" error** - Only fetches 1 JSON file
- **No wiki CORS issues** - Fetch happens server-side
- **Not tied to wiki** - Your API, your branding
- **Fast responses** - 1-hour cache = instant responses
- **All creatures** - No artificial limits
- **Auto-updates** - CRON keeps data fresh

### ❌ Previous Approach Issues:
- Fetched each creature individually = 200+ subrequests = ERROR
- Hit Cloudflare's 50 subrequest limit
- Slow first requests
- Directly tied to wiki availability

## 📝 Example API Response

```json
{
  "count": 150,
  "creatures": [
    {
      "name": "Aereis",
      "tier": 3,
      "diet": "Carnivore",
      "type": "Flier",
      "stats": {
        "health": 2150,
        "weight": 2300,
        "damage": 260,
        "speed_walk": 12.5,
        "speed_sprint": 18.0,
        "speed_fly": 25.0,
        "bite_cooldown": 0.8,
        "growth_time": "26min"
      },
      "abilities": ["Ambush", "Grab", "Glide"],
      "nightvision": "3/3 (Good)"
    }
  ]
}
```

**Notice:** Clean, professional API response. No wiki branding!

## 🐛 Troubleshooting

### "Setup warning" message
→ Update `DATA_SOURCE_URL` or paste data into `EMBEDDED_DATA`

### Empty arrays ({count:0})
→ Your JSON file is not accessible or malformed
→ Test your DATA_SOURCE_URL in a browser
→ Validate JSON at jsonlint.com

### "Failed to fetch data"
→ Check that DATA_SOURCE_URL is publicly accessible
→ Make sure GitHub repo is public (not private)

### CRON not working
→ Verify `[triggers]` section in wrangler.toml
→ Check Cloudflare dashboard → Workers → Triggers tab

### Rate limit errors
→ Increase `RATE_LIMIT_REQUESTS` in worker.js
→ Or set `TESTING_MODE = true` to disable

## 💡 Pro Tips

**1. Use GitHub for easy updates:**
- Edit data.json directly on GitHub
- Worker auto-refreshes within 1 hour
- No redeployment needed!

**2. Monitor your API:**
```bash
# Watch real-time logs
wrangler tail

# View analytics
# Cloudflare Dashboard → Workers → Analytics
```

**3. Customize cache duration:**
```javascript
const CACHE_DURATION = 3600; // 1 hour (adjust as needed)
```

**4. Remove wiki URLs from responses:**
```javascript
// In worker.js, modify creature responses
const cleanCreature = {
  ...creature,
  url: undefined  // Remove wiki URL completely
};
```

## 🎨 Next Steps

1. ✅ Deploy the worker
2. ✅ Test endpoints
3. ✅ Choose data source (GitHub or embedded)
4. ✅ Update data
5. ✅ Set production security settings
6. ✅ Use in your game/website!

## 📚 Quick Reference

```bash
# Deploy
wrangler deploy

# Watch logs
wrangler tail

# Test locally
wrangler dev

# Force refresh
curl https://your-worker.workers.dev/api/refresh
```

---

**You now have a complete, professional API that bypasses the wiki entirely!** 🎉
