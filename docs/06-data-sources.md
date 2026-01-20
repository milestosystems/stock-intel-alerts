# Data Sources

## Overview

This document compares market data providers and specifies which fields are required for each workflow.

**Design Principle:** The system is provider-agnostic. Schemas define what data is needed, not where it comes from. You can swap providers without changing workflow logic.

---

## Required Data Categories

| Category | Used For | Priority |
|----------|----------|----------|
| Quotes (price/volume) | Alerts, Intel Cards | **Required** |
| Company fundamentals | Intel Cards, risk scoring | Recommended |
| News headlines | Alerts, Intel Cards | Recommended |
| SEC filings | Alerts, Intel Cards | Recommended |
| Earnings calendar | Alerts, Daily Briefing | Recommended |
| Analyst ratings | Intel Cards | Optional |
| Insider transactions | Alerts, Intel Cards | Optional |

---

## Provider Comparison

### Quotes + Fundamentals

| Provider | Free Tier | Rate Limits | Notes |
|----------|-----------|-------------|-------|
| **Polygon.io** | 5 calls/min | Generous | Good REST API, websocket available |
| **Finnhub** | 60 calls/min | Good | Includes basic news |
| **Alpha Vantage** | 5 calls/min | Slow | Free but limited |
| **Twelve Data** | 800 calls/day | Moderate | Good value paid tiers |
| **IEX Cloud** | Pay-as-you-go | Flexible | Quality data, clear pricing |
| **Yahoo Finance** | Unofficial | Unreliable | Not recommended for production |

**Recommendation:** Start with Polygon.io (free tier) or Finnhub. Upgrade if rate limits become an issue.

### News

| Provider | Free Tier | Coverage | Notes |
|----------|-----------|----------|-------|
| **Finnhub** | Included | Market news | Basic but functional |
| **Benzinga** | Paid only | Comprehensive | Best for serious use |
| **NewsAPI** | 100 calls/day | General news | Not finance-specific |
| **Polygon.io** | Included | Market news | With paid tiers |

**Recommendation:** Start with Finnhub's included news, upgrade to Benzinga if quality matters.

### SEC Filings

| Provider | Cost | Notes |
|----------|------|-------|
| **SEC EDGAR** | Free | Official source, requires User-Agent header |
| **sec-api.io** | Freemium | Easier API, parsed data |

**Recommendation:** Use SEC EDGAR directly. It's free and authoritative.

---

## Minimum Fields Required

### For Market Pulse (Alerts)

```json
{
  "symbol": "AAPL",
  "latestPrice": 197.45,
  "previousClose": 195.20,
  "change": 2.25,
  "changePercent": 0.0115,
  "volume": 45000000,
  "avgVolume": 50000000,
  "latestTime": "2026-01-20T09:45:00-05:00"
}
```

### For Intel Cards

All of the above, plus:

```json
{
  "companyName": "Apple Inc.",
  "marketCap": 3100000000000,
  "peRatio": 32.5,
  "week52High": 204.12,
  "week52Low": 164.08,
  "nextEarningsDate": "2026-01-30",
  "news": [
    {
      "headline": "...",
      "source": "Reuters",
      "datetime": "2026-01-20T08:30:00Z",
      "url": "..."
    }
  ],
  "filings": [
    {
      "formType": "8-K",
      "filedAt": "2026-01-18",
      "accessionNumber": "...",
      "url": "..."
    }
  ]
}
```

---

## SEC EDGAR Integration

### User-Agent Requirement

SEC requires a User-Agent header with your name and email:

```
User-Agent: stock-intel-alerts (your-email@example.com)
```

### Useful Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/cgi-bin/browse-edgar?action=getcompany&CIK={cik}&type=8-K&count=5&output=atom` | Recent filings by form type |
| `/cgi-bin/browse-edgar?action=getcompany&CIK={cik}&type=4&count=10&output=atom` | Insider transactions |

### CIK Lookup

Ticker to CIK mapping: `https://www.sec.gov/cgi-bin/browse-edgar?company=&CIK={ticker}&type=&owner=include&count=1&action=getcompany`

---

## API Key Management

See `docs/07-security-keys.md` for credential storage best practices.

**Quick rules:**
- Never commit API keys to the repository
- - Use n8n credentials store
  - - Rotate keys if exposed
    - - Use environment variables for local development
     
      - ---

      ## Rate Limit Strategies

      1. **Cache aggressively** — Quote data is valid for 1-5 minutes
      2. 2. **Batch requests** — Many providers support multi-symbol queries
         3. 3. **Stagger fetches** — Don't hit all tickers simultaneously
            4. 4. **Respect limits** — Implement exponential backoff on 429 errors
              
               5. ---
              
               6. ## Provider Setup Checklist
              
               7. - [ ] Choose primary quote provider
                  - [ ] - [ ] Sign up and get API key
                  - [ ] - [ ] Test API with single ticker
                  - [ ] - [ ] Configure n8n credentials
                  - [ ] - [ ] Set up SEC EDGAR User-Agent
                  - [ ] - [ ] (Optional) Configure news provider
                  - [ ] - [ ] (Optional) Configure earnings calendar source
