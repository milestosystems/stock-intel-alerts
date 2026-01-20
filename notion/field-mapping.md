# Notion Field Mapping

**Purpose:** Map JSON schema fields to Notion database properties for consistent n8n wiring.

**Last Updated:** 2026-01-20

---

## Watchlist Database

| Schema Field | Notion Property | Type | Notes |
|--------------|-----------------|------|-------|
| `ticker` | Ticker | Title | Uppercase symbol (e.g., AAPL) |
| `companyName` | Company Name | Text | Full company name |
| `status` | Status | Select | Options: active, watching, paused, archived |
| `alertPriority` | Alert Priority | Select | Options: high, medium, low, muted |
| `thesis` | Thesis | Text | Investment thesis / notes |
| `addedReason` | Added Reason | Text | Why added to watchlist |
| `tags` | Tags | Multi-select | Custom tags (e.g., earnings-play, swing) |
| `addedAt` | Added At | Date | When ticker was added |
| `lastAlertAt` | Last Alert | Date | Timestamp of last alert |
| `lastIntelCardAt` | Last Intel Card | Date | Timestamp of last Intel Card |
| `alertCount` | Alert Count | Number | Total alerts generated |
| `notionPageId` | — | — | Auto-generated, use for API references |

**Notion Database ID:** `[FILL IN AFTER CREATING]`

---

## Intel Cards Database

| Schema Field | Notion Property | Type | Notes |
|--------------|-----------------|------|-------|
| `cardId` | — | — | Use Notion page ID |
| `ticker` | Ticker | Title | Links to Watchlist |
| `generatedAt` | Generated At | Date | With time |
| `snapshot.price` | Price | Number | Current price |
| `snapshot.changePercent` | Change % | Number | Day change % |
| `snapshot.volume` | Volume | Number | Today's volume |
| `snapshot.volumeRatio` | Volume Ratio | Number | vs 20-day avg |
| `catalysts[0].type` | Next Catalyst Type | Select | earnings, fda, etc. |
| `catalysts[0].date` | Next Catalyst Date | Date | Upcoming event date |
| `risk.riskScore` | Risk Score | Number | 0-100 |
| `risk.flags` | Risk Flags | Text | Comma-separated or bullets |
| `synthesis.headline` | Headline | Text | One-line summary |
| `synthesis.summary` | Summary | Text | 2-3 sentence overview |
| `synthesis.whatToWatch` | What to Watch | Text | Key items to monitor |
| `sources.quoteProvider` | Quote Source | Text | Data attribution |
| `sources.quoteTimestamp` | Quote Timestamp | Date | Data freshness |

**Notion Database ID:** `[FILL IN AFTER CREATING]`

---

## Journal Database (Alerts)

| Schema Field | Notion Property | Type | Notes |
|--------------|-----------------|------|-------|
| `alertId` | — | — | Use Notion page ID |
| `ticker` | Ticker | Text | Or Relation to Watchlist |
| `alertType` | Alert Type | Select | price_spike, volume_surge, etc. |
| `severity` | Severity | Select | low, medium, high, critical |
| `headline` | Headline | Title | Short alert headline |
| `whySeeing` | Why Seeing | Text | Explanation of trigger |
| `triggeredAt` | Triggered At | Date | With time |
| `dataTimestamp` | Data Timestamp | Date | When underlying data was fetched |
| `source` | Source | Text | Data provider name |
| `cooldownKey` | Cooldown Key | Text | For deduplication (ticker:type:date) |
| `cooldownUntil` | Cooldown Until | Date | When similar alerts resume |
| `acknowledged` | Acknowledged | Checkbox | User dismissed/viewed |
| `actionTaken` | Action Taken | Select | none, watched, researched, traded |
| `deliveryChannels` | Delivered To | Multi-select | discord, telegram, email |

**Notion Database ID:** `[FILL IN AFTER CREATING]`

---

## n8n Mapping Notes

### Best Practices

1. **Create a mapper Code node** that transforms raw API responses into schema-compliant JSON
2. 2. **Validate schema JSON** before writing to Notion
   3. 3. **Use consistent date formats** — ISO 8601 for all timestamps
      4. 4. **Store Notion page IDs** in your schema objects for updates/references
        
         5. ### Example Mapper Pattern (n8n Code Node)
        
         6. ```javascript
            // Transform quote API response to schema format
            const quote = $input.first().json;

            return {
              ticker: quote.symbol,
              snapshot: {
                price: quote.latestPrice,
                changePercent: quote.changePercent * 100,
                volume: quote.volume,
                volumeRatio: quote.volume / quote.avgTotalVolume
              },
              dataTimestamp: new Date().toISOString(),
              source: "IEX Cloud" // or your provider
            };
            ```

            ### Cooldown Key Format

            Use this pattern for deduplication:
            ```
            {ticker}:{alertType}:{YYYY-MM-DD}
            ```

            Example: `AAPL:price_spike:2026-01-20`

            ---

            ## Setup Checklist

            - [ ] Create Watchlist database in Notion
            - [ ] - [ ] Create Intel Cards database in Notion
            - [ ] - [ ] Create Journal database in Notion
            - [ ] - [ ] Fill in Database IDs above
            - [ ] - [ ] Create Notion integration and get API token
            - [ ] - [ ] Add token to n8n credentials
            - [ ] - [ ] Test read/write with each database
