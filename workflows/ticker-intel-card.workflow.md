# Ticker Intel Card Workflow

**Workflow Type**: On-Demand (Webhook Triggered)  
**Trigger**: Discord/Telegram command or webhook request  
**Purpose**: Generate comprehensive "background check" Intel Card for any ticker

## Overview

The Intel Card workflow is triggered on-demand when a user requests analysis of a specific ticker. It aggregates data from multiple sources, calculates risk scores, generates an AI synthesis, and delivers the formatted card to the user.

## Workflow Diagram

```
┌─────────────────┐     ┌─────────────────┐
│ Webhook Trigger │────►│ Parse Ticker    │
│ /intel {ticker} │     │ from Request    │
└─────────────────┘     └────────┬────────┘
                                 │
                    ┌────────────┴────────────┐
                    │     PARALLEL FETCH       │
                    ├─────────────────────────┤
    ┌───────────────┼───────────────┐         │
    ▼               ▼               ▼         │
┌─────────┐   ┌─────────┐   ┌─────────┐      │
│ Stock   │   │ News    │   │ SEC     │      │
│ Quote + │   │ Headlines│   │ Filings │      │
│ Company │   │         │   │         │      │
└────┬────┘   └────┬────┘   └────┬────┘      │
     │             │             │            │
     └─────────────┴─────────────┘            │
                    │                         │
                    ▼                         │
           ┌───────────────┐                  │
           │ Merge All Data│◄─────────────────┘
           └───────┬───────┘
                   │
                   ▼
           ┌───────────────┐
           │ Calculate Risk│
           │ Score & Flags │
           └───────┬───────┘
                   │
                   ▼
           ┌───────────────┐
           │ AI Synthesis  │
           │ (LLM Summary) │
           └───────┬───────┘
                   │
                   ▼
           ┌───────────────┐
           │ Format Intel  │
           │ Card Message  │
           └───────┬───────┘
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
┌──────────────┐    ┌──────────────┐
│ Send to User │    │ Save to      │
│ (Discord/    │    │ Notion DB    │
│ Telegram)    │    │              │
└──────────────┘    └──────────────┘
```

## n8n Node-by-Node Specification

### Node 1: Webhook Trigger

**Node Type**: `n8n-nodes-base.webhook`

**Configuration**:
```json
{
  "httpMethod": "POST",
  "path": "intel-card",
  "responseMode": "lastNode",
  "options": {
    "rawBody": false
  }
}
```

**Expected Input**:
```json
{
  "ticker": "AAPL",
  "user_id": "discord_user_123",
  "channel": "discord"
}
```

**Alternative: Discord Trigger**:
```json
{
  "triggerType": "messageCreate",
  "messageFilter": "startsWith",
  "filterValue": "/intel"
}
```

---

### Node 2: Parse and Validate Ticker

**Node Type**: `n8n-nodes-base.set`

**Purpose**: Extract ticker symbol and validate

**Configuration**:
```javascript
// Extract ticker from request
let ticker = $json.ticker || '';

// If from Discord message, parse from content
if ($json.content && $json.content.startsWith('/intel')) {
  ticker = $json.content.split(' ')[1]?.toUpperCase() || '';
}

// Validate ticker format (1-5 uppercase letters)
const isValid = /^[A-Z]{1,5}$/.test(ticker);

if (!isValid) {
  throw new Error('Invalid ticker symbol. Use 1-5 letter symbol (e.g., AAPL)');
}

return {
  ticker,
  userId: $json.user_id || $json.author?.id,
  channel: $json.channel || 'webhook',
  requestTime: new Date().toISOString()
};
```

---

### Node 3: Parallel Data Fetch (Execute in Parallel)

Use **Split in Batches** or **Execute Workflow** with parallel execution.

#### Node 3a: Get Stock Quote + Company Info

**Node Type**: `n8n-nodes-base.httpRequest`

**Alpha Vantage - Quote**:
```json
{
  "method": "GET",
  "url": "https://www.alphavantage.co/query",
  "qs": {
    "function": "GLOBAL_QUOTE",
    "symbol": "={{ $json.ticker }}",
    "apikey": "={{ $credentials.alphaVantageApiKey }}"
  }
}
```

**Alpha Vantage - Company Overview**:
```json
{
  "method": "GET",
  "url": "https://www.alphavantage.co/query",
  "qs": {
    "function": "OVERVIEW",
    "symbol": "={{ $json.ticker }}",
    "apikey": "={{ $credentials.alphaVantageApiKey }}"
  }
}
```

#### Node 3b: Get News Headlines

**Node Type**: `n8n-nodes-base.httpRequest`

**Finnhub News**:
```json
{
  "method": "GET",
  "url": "https://finnhub.io/api/v1/company-news",
  "qs": {
    "symbol": "={{ $json.ticker }}",
    "from": "={{ new Date(Date.now() - 7*24*60*60*1000).toISOString().split('T')[0] }}",
    "to": "={{ new Date().toISOString().split('T')[0] }}",
    "token": "={{ $credentials.finnhubApiKey }}"
  }
}
```

**Alternative - NewsAPI**:
```json
{
  "method": "GET",
  "url": "https://newsapi.org/v2/everything",
  "qs": {
    "q": "={{ $json.ticker + ' stock' }}",
    "sortBy": "publishedAt",
    "pageSize": 5,
    "apiKey": "={{ $credentials.newsApiKey }}"
  }
}
```

#### Node 3c: Get SEC Filings

**Node Type**: `n8n-nodes-base.httpRequest`

**SEC EDGAR API**:
```json
{
  "method": "GET",
  "url": "https://data.sec.gov/submissions/CIK{{ $json.cik }}.json",
  "headers": {
    "User-Agent": "StockIntelBot contact@example.com"
  }
}
```

**Note**: Need to first lookup CIK from ticker using SEC company tickers endpoint.

---

### Node 4: Merge All Data

**Node Type**: `n8n-nodes-base.merge`

**Configuration**:
```json
{
  "mode": "append"
}
```

Then use **Set** node to combine into single object:

```javascript
// Combine all fetched data into Intel Card structure
const quote = $node["Get Quote"].json["Global Quote"];
const company = $node["Get Company"].json;
const news = $node["Get News"].json.articles || $node["Get News"].json;
const filings = $node["Get Filings"].json.filings?.recent || [];

return {
  ticker: $json.ticker,
  companyName: company.Name || 'Unknown',

  // Price data
  price: parseFloat(quote["05. price"]),
  open: parseFloat(quote["02. open"]),
  high: parseFloat(quote["03. high"]),
  low: parseFloat(quote["04. low"]),
  previousClose: parseFloat(quote["08. previous close"]),
  change: parseFloat(quote["09. change"]),
  changePercent: parseFloat(quote["10. change percent"]?.replace('%', '')),
  volume: parseInt(quote["06. volume"]),

  // Company data
  marketCap: parseFloat(company.MarketCapitalization),
  peRatio: parseFloat(company.PERatio),
  eps: parseFloat(company.EPS),
  week52High: parseFloat(company["52WeekHigh"]),
  week52Low: parseFloat(company["52WeekLow"]),
  sector: company.Sector,
  industry: company.Industry,

  // Earnings
  nextEarningsDate: company.DividendDate, // Use earnings calendar API for accurate date

  // News (top 5)
  headlines: news.slice(0, 5).map(n => ({
    title: n.headline || n.title,
    source: n.source?.name || n.source,
    url: n.url,
    publishedAt: n.datetime || n.publishedAt
  })),

  // Filings (last 3)
  recentFilings: filings.slice(0, 3).map((f, i) => ({
    type: filings.form?.[i],
    date: filings.filingDate?.[i],
    url: `https://www.sec.gov/Archives/edgar/data/${company.CIK}/${filings.accessionNumber?.[i]}`
  })),

  dataTimestamp: new Date().toISOString()
};
```

---

### Node 5: Calculate Risk Score

**Node Type**: `n8n-nodes-base.function`

**Configuration**:
```javascript
const data = $json;
let riskScore = 5; // Start at neutral
const riskFlags = [];

// Volatility check (price near 52-week extremes)
const priceRange = data.week52High - data.week52Low;
const pricePosition = (data.price - data.week52Low) / priceRange;

if (pricePosition >= 0.9) {
  riskScore += 1;
  riskFlags.push('Price Near 52W High');
}
if (pricePosition <= 0.1) {
  riskScore += 1;
  riskFlags.push('Price Near 52W Low');
}

// Volatility - large intraday range
const intradayRange = (data.high - data.low) / data.open;
if (intradayRange > 0.03) { // > 3% range
  riskScore += 1;
  riskFlags.push('High Volatility');
}

// P/E ratio check
if (data.peRatio > 50) {
  riskScore += 1;
  riskFlags.push('High P/E Ratio');
}
if (data.peRatio < 0) {
  riskScore += 2;
  riskFlags.push('Negative Earnings');
}

// Recent significant move
if (Math.abs(data.changePercent) > 5) {
  riskScore += 1;
  riskFlags.push('Large Recent Move');
}

// Earnings imminent (within 7 days)
// Would need earnings calendar data
if (data.earningsInDays && data.earningsInDays <= 7) {
  riskScore += 1;
  riskFlags.push('Earnings Imminent');
}

// Cap score at 10
riskScore = Math.min(10, Math.max(1, riskScore));

return {
  ...data,
  riskScore,
  riskFlags,
  riskLevel: riskScore <= 3 ? 'Low' : riskScore <= 6 ? 'Medium' : 'High'
};
```

---

### Node 6: AI Synthesis

**Node Type**: `n8n-nodes-base.openAi` or `n8n-nodes-base.httpRequest`

**OpenAI Configuration**:
```json
{
  "resource": "chat",
  "operation": "create",
  "model": "gpt-4o-mini",
  "messages": {
    "values": [
      {
        "role": "system",
        "content": "You are a stock market analyst providing brief, factual summaries. Never give buy/sell recommendations. Focus on risk awareness and key data points. Keep responses under 100 words."
      },
      {
        "role": "user",
        "content": "={{ 'Summarize this stock data for ' + $json.ticker + ':\\n' + JSON.stringify({price: $json.price, change: $json.changePercent + '%', marketCap: $json.marketCap, peRatio: $json.peRatio, riskScore: $json.riskScore, riskFlags: $json.riskFlags, recentNews: $json.headlines.map(h => h.title)}, null, 2) }}"
      }
    ]
  },
  "options": {
    "maxTokens": 200,
    "temperature": 0.3
  }
}
```

**Alternative - Claude API**:
```json
{
  "method": "POST",
  "url": "https://api.anthropic.com/v1/messages",
  "headers": {
    "x-api-key": "={{ $credentials.anthropicApiKey }}",
    "anthropic-version": "2023-06-01",
    "Content-Type": "application/json"
  },
  "body": {
    "model": "claude-3-haiku-20240307",
    "max_tokens": 200,
    "messages": [
      {
        "role": "user",
        "content": "Provide a brief (under 100 words), factual summary of this stock. No buy/sell recommendations..."
      }
    ]
  }
}
```

---

### Node 7: Format Intel Card Message

**Node Type**: `n8n-nodes-base.set`

**Configuration**:
```javascript
const d = $json;
const aiSummary = $node["AI Synthesis"].json.choices?.[0]?.message?.content ||
                  $node["AI Synthesis"].json.content?.[0]?.text ||
                  'Summary unavailable';

// Format large numbers
const formatNum = (n) => {
  if (n >= 1e12) return (n/1e12).toFixed(2) + 'T';
  if (n >= 1e9) return (n/1e9).toFixed(2) + 'B';
  if (n >= 1e6) return (n/1e6).toFixed(2) + 'M';
  return n?.toLocaleString() || 'N/A';
};

const direction = d.changePercent >= 0 ? '📈' : '📉';
const riskEmoji = d.riskLevel === 'Low' ? '🟢' : d.riskLevel === 'Medium' ? '🟡' : '🔴';

const card = `
**📊 INTEL CARD: ${d.ticker}**
_${d.companyName} | ${d.sector}_

━━━━━━━━━━━━━━━━━━
**💰 SNAPSHOT**
${direction} Price: $${d.price.toFixed(2)} (${d.changePercent >= 0 ? '+' : ''}${d.changePercent.toFixed(2)}%)
📊 Day Range: $${d.low.toFixed(2)} - $${d.high.toFixed(2)}
🏢 Market Cap: $${formatNum(d.marketCap)}
📈 P/E Ratio: ${d.peRatio || 'N/A'}

**📉 TREND CONTEXT**
52W High: $${d.week52High?.toFixed(2) || 'N/A'}
52W Low: $${d.week52Low?.toFixed(2) || 'N/A'}
Volume: ${formatNum(d.volume)}

**⚠️ RISK ASSESSMENT**
${riskEmoji} Risk Score: ${d.riskScore}/10 (${d.riskLevel})
${d.riskFlags.length > 0 ? 'Flags: ' + d.riskFlags.join(', ') : 'No major flags detected'}

**📰 RECENT NEWS**
${d.headlines.slice(0, 3).map((h, i) => `${i+1}. ${h.title.substring(0, 60)}...`).join('\\n')}

**📄 LATEST FILINGS**
${d.recentFilings.slice(0, 2).map(f => `• ${f.type} (${f.date})`).join('\\n') || 'No recent filings'}

**🤖 AI SUMMARY**
${aiSummary}

━━━━━━━━━━━━━━━━━━
⏱️ Data as of: ${new Date(d.dataTimestamp).toLocaleString()}
⚠️ _Not financial advice. Data may be delayed. Verify before acting._
`;

return {
  ...d,
  aiSummary,
  formattedCard: card
};
```

---

### Node 8: Send to User

**Node Type**: `n8n-nodes-base.discord` or `n8n-nodes-base.telegram`

**Discord Configuration**:
```json
{
  "resource": "webhook",
  "operation": "send",
  "webhookUri": "={{ $json.webhookUrl || $credentials.discordWebhook }}",
  "content": "={{ $json.formattedCard }}",
  "options": {
    "username": "Stock Intel Bot"
  }
}
```

**Telegram Configuration**:
```json
{
  "resource": "message",
  "operation": "sendMessage",
  "chatId": "={{ $json.chatId || $credentials.telegramChatId }}",
  "text": "={{ $json.formattedCard }}",
  "parseMode": "Markdown"
}
```

---

### Node 9: Save to Notion

**Node Type**: `n8n-nodes-base.notion`

**Configuration**:
```json
{
  "resource": "databasePage",
  "operation": "create",
  "databaseId": "={{ $credentials.notionIntelCardsDb }}",
  "properties": {
    "Title": {
      "title": [{"text": {"content": "={{ $json.ticker + ' - ' + new Date().toISOString().split('T')[0] }}"}}]
    },
    "Ticker": {
      "rich_text": [{"text": {"content": "={{ $json.ticker }}"}}]
    },
    "Generated At": {
      "date": {"start": "={{ $json.dataTimestamp }}"}
    },
    "Trigger": {
      "select": {"name": "On-Demand"}
    },
    "Current Price": {
      "number": "={{ $json.price }}"
    },
    "Day Change %": {
      "number": "={{ $json.changePercent }}"
    },
    "Market Cap": {
      "number": "={{ $json.marketCap }}"
    },
    "Risk Score": {
      "number": "={{ $json.riskScore }}"
    },
    "Risk Flags": {
      "multi_select": "={{ $json.riskFlags.map(f => ({name: f})) }}"
    },
    "AI Summary": {
      "rich_text": [{"text": {"content": "={{ $json.aiSummary }}"}}]
    },
    "Headlines JSON": {
      "rich_text": [{"text": {"content": "={{ JSON.stringify($json.headlines) }}"}}]
    }
  }
}
```

---

## Error Handling

### Invalid Ticker
```javascript
if (!validTickers.includes(ticker)) {
  return {
    error: true,
    message: `Ticker "${ticker}" not found. Please check the symbol and try again.`
  };
}
```

### API Failures
- Use **Try/Catch** wrapper around each API call
- - Return partial data if some sources fail
  - - Include "Data unavailable" for missing sections
   
    - ### Rate Limiting
    - - Add 1-second delay between API calls
      - - Queue requests if multiple users trigger simultaneously
       
        - ---

        ## Configuration Variables

        | Variable | Description |
        |----------|-------------|
        | `ALPHA_VANTAGE_KEY` | Stock quote API key |
        | `FINNHUB_API_KEY` | News API key |
        | `OPENAI_API_KEY` | AI synthesis API key |
        | `NOTION_INTEL_CARDS_DB` | Notion database ID |
        | `DISCORD_WEBHOOK` | Discord delivery webhook |

        ---

        ## Testing Checklist

        - [ ] Webhook receives requests correctly
        - [ ] - [ ] Ticker validation works
        - [ ] - [ ] All API calls return data
        - [ ] - [ ] Risk score calculates correctly
        - [ ] - [ ] AI summary generates without errors
        - [ ] - [ ] Message formats properly for Discord/Telegram
        - [ ] - [ ] Notion entry is created
        - [ ] - [ ] Error messages are user-friendly
        - [ ] - [ ] Response time is under 30 seconds
       
        - [ ] ---
       
        - [ ] ## Estimated Implementation Time
       
        - [ ] | Task | Duration |
        - [ ] |------|----------|
        - [ ] | Webhook setup | 15 min |
        - [ ] | Stock data integration | 45 min |
        - [ ] | News integration | 30 min |
        - [ ] | SEC filings integration | 45 min |
        - [ ] | Risk scoring logic | 30 min |
        - [ ] | AI synthesis setup | 30 min |
        - [ ] | Message formatting | 30 min |
        - [ ] | Notion integration | 30 min |
        - [ ] | Testing and debugging | 60 min |
        - [ ] | **Total** | **~5 hours** |
