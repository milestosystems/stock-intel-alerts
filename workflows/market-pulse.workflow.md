# Market Pulse Workflow

**Workflow Type**: Scheduled (Cron)  
**Trigger**: Every 15 minutes during market hours  
**Purpose**: Scan watchlist for alert conditions and send notifications

## Overview

The Market Pulse workflow runs periodically during market hours to check each watched ticker against configured alert rules. When conditions are met and cooldowns have expired, it sends alerts to the configured delivery channels.

## Workflow Diagram

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────────┐
│   Cron      │────►│  Get Watchlist  │────►│  For Each Ticker │
│  Trigger    │     │  from Notion    │     │                  │
└─────────────┘     └─────────────────┘     └────────┬─────────┘
                                                     │
                    ┌────────────────────────────────┘
                    ▼
┌──────────────────────────────────────────────────────────────┐
│                    FOR EACH TICKER                            │
├──────────────────────────────────────────────────────────────┤
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐ │
│  │ Fetch Quote  │────►│ Check Rules  │────►│ Check        │ │
│  │ Data         │     │ (IF nodes)   │     │ Cooldowns    │ │
│  └──────────────┘     └──────────────┘     └──────┬───────┘ │
│                                                    │         │
│                       ┌────────────────────────────┘         │
│                       ▼                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              IF ALERT TRIGGERED                       │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  │   │
│  │  │ Format      │──│ Send Alert  │──│ Log to       │  │   │
│  │  │ Message     │  │ (Discord/   │  │ Notion       │  │   │
│  │  │             │  │  Telegram)  │  │ Journal      │  │   │
│  │  └─────────────┘  └─────────────┘  └──────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

## n8n Node-by-Node Specification

### Node 1: Cron Trigger

**Node Type**: `n8n-nodes-base.cron`

**Configuration**:
```json
{
  "triggerTimes": {
    "item": [
      {
        "mode": "custom",
        "cronExpression": "*/15 9-16 * * 1-5"
      }
    ]
  }
}
```

**Notes**:
- Runs every 15 minutes
- - Only Monday-Friday (1-5)
  - - Only 9 AM to 4 PM (market hours, adjust for your timezone)
    - - Consider adding market holiday checks
     
      - ---

      ### Node 2: Check Market Hours (Optional)

      **Node Type**: `n8n-nodes-base.if`

      **Purpose**: Skip execution on market holidays or outside regular hours

      **Configuration**:
      ```javascript
      // Expression to check if market is open
      const now = new Date();
      const hour = now.getUTCHours() - 5; // Convert to ET
      const day = now.getUTCDay();
      const isWeekday = day >= 1 && day <= 5;
      const isMarketHours = hour >= 9 && hour < 16;

      // Add holiday check here
      const holidays = ['2024-01-01', '2024-01-15', /* ... */];
      const today = now.toISOString().split('T')[0];
      const isHoliday = holidays.includes(today);

      return isWeekday && isMarketHours && !isHoliday;
      ```

      ---

      ### Node 3: Get Watchlist from Notion

      **Node Type**: `n8n-nodes-base.notion`

      **Operation**: Get Database Items

      **Configuration**:
      ```json
      {
        "resource": "databasePage",
        "operation": "getAll",
        "databaseId": "={{ $credentials.notionWatchlistDb }}",
        "returnAll": true,
        "filterType": "manual",
        "filters": {
          "conditions": [
            {
              "key": "Status",
              "condition": "equals",
              "value": "Active"
            }
          ]
        }
      }
      ```

      **Output**: Array of watchlist items with ticker symbols

      ---

      ### Node 4: Split Into Batches

      **Node Type**: `n8n-nodes-base.splitInBatches`

      **Purpose**: Process tickers one at a time to respect API rate limits

      **Configuration**:
      ```json
      {
        "batchSize": 1,
        "options": {}
      }
      ```

      ---

      ### Node 5: Get Stock Quote

      **Node Type**: `n8n-nodes-base.httpRequest`

      **Configuration** (Alpha Vantage example):
      ```json
      {
        "method": "GET",
        "url": "https://www.alphavantage.co/query",
        "qs": {
          "function": "GLOBAL_QUOTE",
          "symbol": "={{ $json.properties.Ticker.title[0].plain_text }}",
          "apikey": "={{ $credentials.alphaVantageApiKey }}"
        },
        "options": {
          "timeout": 10000
        }
      }
      ```

      **Alternative** (Yahoo Finance via RapidAPI):
      ```json
      {
        "method": "GET",
        "url": "https://yahoo-finance15.p.rapidapi.com/api/yahoo/qu/quote/{{ $json.ticker }}",
        "headers": {
          "X-RapidAPI-Key": "={{ $credentials.rapidApiKey }}",
          "X-RapidAPI-Host": "yahoo-finance15.p.rapidapi.com"
        }
      }
      ```

      ---

      ### Node 6: Parse Quote Data

      **Node Type**: `n8n-nodes-base.set`

      **Purpose**: Normalize quote data into standard format

      **Configuration**:
      ```javascript
      // For Alpha Vantage response
      const quote = $json["Global Quote"];
      return {
        ticker: quote["01. symbol"],
        price: parseFloat(quote["05. price"]),
        open: parseFloat(quote["02. open"]),
        high: parseFloat(quote["03. high"]),
        low: parseFloat(quote["04. low"]),
        volume: parseInt(quote["06. volume"]),
        previousClose: parseFloat(quote["08. previous close"]),
        change: parseFloat(quote["09. change"]),
        changePercent: parseFloat(quote["10. change percent"].replace('%', '')),
        timestamp: new Date().toISOString()
      };
      ```

      ---

      ### Node 7: Check Price Move Rule

      **Node Type**: `n8n-nodes-base.if`

      **Purpose**: Check if price change exceeds threshold

      **Configuration**:
      ```javascript
      const changePercent = Math.abs($json.changePercent);
      const threshold = 3; // Default 3%, load from config
      return changePercent >= threshold;
      ```

      **True Branch**: Proceed to cooldown check
      **False Branch**: Continue to next rule

      ---

      ### Node 8: Check Volume Spike Rule

      **Node Type**: `n8n-nodes-base.if`

      **Purpose**: Check if volume exceeds average

      **Note**: Requires historical average volume (fetch separately or store in watchlist)

      **Configuration**:
      ```javascript
      const currentVolume = $json.volume;
      const avgVolume = $json.avgVolume20d || 1000000; // From watchlist or separate fetch
      const multiplier = 2; // Default 2x
      return currentVolume >= (avgVolume * multiplier);
      ```

      ---

      ### Node 9: Check Cooldown

      **Node Type**: `n8n-nodes-base.notion`

      **Operation**: Query Database

      **Purpose**: Check if similar alert was sent recently

      **Configuration**:
      ```json
      {
        "resource": "databasePage",
        "operation": "getAll",
        "databaseId": "={{ $credentials.notionJournalDb }}",
        "filterType": "manual",
        "filters": {
          "conditions": [
            {
              "key": "Cooldown Key",
              "condition": "equals",
              "value": "={{ 'price_move:' + $json.ticker }}"
            },
            {
              "key": "Timestamp",
              "condition": "after",
              "value": "={{ new Date(Date.now() - 4*60*60*1000).toISOString() }}"
            }
          ]
        }
      }
      ```

      **Logic**: If results found, cooldown not expired → skip alert

      ---

      ### Node 10: Format Alert Message

      **Node Type**: `n8n-nodes-base.set`

      **Purpose**: Build the alert message using template

      **Configuration**:
      ```javascript
      const ticker = $json.ticker;
      const changePercent = $json.changePercent;
      const direction = changePercent >= 0 ? "up" : "down";
      const emoji = changePercent >= 0 ? "📈" : "📉";

      const message = `🔔 **${ticker}** moved ${direction} ${Math.abs(changePercent).toFixed(2)}%

      ${emoji} Current: $${$json.price.toFixed(2)} | Open: $${$json.open.toFixed(2)}
      📊 Day Range: $${$json.low.toFixed(2)} - $${$json.high.toFixed(2)}
      📦 Volume: ${($json.volume / 1000000).toFixed(2)}M

      **Why this alert?**
      ${ticker} exceeded your 3% price move threshold.
      This is an informational alert about significant price action.

      ⏱️ Data as of ${$json.timestamp}

      _Not financial advice. Verify before acting._`;

      return {
        ticker,
        alertType: 'price_move',
        severity: Math.abs(changePercent) >= 5 ? 'high' : 'medium',
        message,
        cooldownKey: `price_move:${ticker}`,
        ...($json)
      };
      ```

      ---

      ### Node 11: Send to Discord

      **Node Type**: `n8n-nodes-base.discord`

      **Configuration**:
      ```json
      {
        "resource": "webhook",
        "operation": "send",
        "webhookUri": "={{ $credentials.discordWebhook }}",
        "content": "={{ $json.message }}",
        "options": {
          "username": "Stock Intel Bot",
          "avatar_url": "https://example.com/bot-avatar.png"
        }
      }
      ```

      **Alternative: Telegram**:
      ```json
      {
        "resource": "message",
        "operation": "sendMessage",
        "chatId": "={{ $credentials.telegramChatId }}",
        "text": "={{ $json.message }}",
        "parseMode": "Markdown"
      }
      ```

      ---

      ### Node 12: Log to Journal

      **Node Type**: `n8n-nodes-base.notion`

      **Operation**: Create Database Page

      **Configuration**:
      ```json
      {
        "resource": "databasePage",
        "operation": "create",
        "databaseId": "={{ $credentials.notionJournalDb }}",
        "properties": {
          "Title": {
            "title": [
              {
                "text": {
                  "content": "={{ $json.ticker + ' moved ' + $json.changePercent.toFixed(1) + '%' }}"
                }
              }
            ]
          },
          "Ticker": {
            "rich_text": [
              {
                "text": {
                  "content": "={{ $json.ticker }}"
                }
              }
            ]
          },
          "Alert Type": {
            "select": {
              "name": "Price Move"
            }
          },
          "Severity": {
            "select": {
              "name": "={{ $json.severity }}"
            }
          },
          "Timestamp": {
            "date": {
              "start": "={{ new Date().toISOString() }}"
            }
          },
          "Message": {
            "rich_text": [
              {
                "text": {
                  "content": "={{ $json.message }}"
                }
              }
            ]
          },
          "Cooldown Key": {
            "rich_text": [
              {
                "text": {
                  "content": "={{ $json.cooldownKey }}"
                }
              }
            ]
          },
          "Channel": {
            "select": {
              "name": "Discord"
            }
          }
        }
      }
      ```

      ---

      ## Error Handling

      ### Rate Limit Handling

      Add a **Wait** node after API calls:
      ```json
      {
        "amount": 1,
        "unit": "seconds"
      }
      ```

      ### API Error Handling

      Wrap HTTP Request nodes with **Error Trigger** node to:
      1. Log errors to a separate error log
      2. 2. Continue processing remaining tickers
         3. 3. Send admin alert if too many failures
           
            4. ### Retry Logic
           
            5. Configure HTTP Request nodes with:
            6. ```json
               {
                 "options": {
                   "retry": {
                     "maxTries": 3,
                     "wait": {
                       "waitBetweenTries": 2000
                     }
                   }
                 }
               }
               ```

               ---

               ## Configuration Variables

               Store these as n8n credentials or environment variables:

               | Variable | Description | Example |
               |----------|-------------|---------|
               | `NOTION_API_KEY` | Notion integration token | `secret_xxx` |
               | `NOTION_WATCHLIST_DB` | Watchlist database ID | `abc123...` |
               | `NOTION_JOURNAL_DB` | Journal database ID | `def456...` |
               | `ALPHA_VANTAGE_KEY` | Stock data API key | `ABCD1234` |
               | `DISCORD_WEBHOOK` | Discord webhook URL | `https://discord.com/api/webhooks/...` |
               | `TELEGRAM_BOT_TOKEN` | Telegram bot token | `123456:ABC...` |
               | `TELEGRAM_CHAT_ID` | Telegram chat ID | `-1001234567890` |

               ---

               ## Testing Checklist

               - [ ] Cron trigger fires at expected times
               - [ ] - [ ] Watchlist is fetched correctly
               - [ ] - [ ] Stock quotes return valid data
               - [ ] - [ ] Price move alerts trigger at threshold
               - [ ] - [ ] Cooldown prevents duplicate alerts
               - [ ] - [ ] Discord/Telegram messages format correctly
               - [ ] - [ ] Journal entries are created
               - [ ] - [ ] Errors are handled gracefully
               - [ ] - [ ] Rate limits are respected
              
               - [ ] ---
              
               - [ ] ## Performance Considerations
              
               - [ ] 1. **Batch Size**: Process 1 ticker at a time to avoid rate limits
               - [ ] 2. **Timeout**: Set 10-second timeout on API calls
               - [ ] 3. **Retry**: Max 3 retries with 2-second wait
               - [ ] 4. **Cooldown**: Minimum 4-hour gap between same-type alerts
               - [ ] 5. **Market Hours**: Skip execution outside trading hours
              
               - [ ] ---
              
               - [ ] ## Estimated Implementation Time
              
               - [ ] | Task | Duration |
               - [ ] |------|----------|
               - [ ] | Basic workflow setup | 30 min |
               - [ ] | Notion integration | 30 min |
               - [ ] | Stock API integration | 45 min |
               - [ ] | Alert rules logic | 45 min |
               - [ ] | Discord/Telegram setup | 30 min |
               - [ ] | Testing and debugging | 60 min |
               - [ ] | **Total** | **~4 hours** |
