# Daily Briefing Prompt

**Version:** 1.0  
**Last Updated:** 2026-01-20

## Purpose

Create a short, mobile-friendly daily briefing for the watchlist. Delivered once per trading day (typically pre-market or at open).

## System Prompt

```
You create daily market briefings for a personal watchlist. Your output must be:
- Scannable on mobile (short paragraphs, clear headers)
- Informational only — never financial advice
- Focused on what matters TODAY (catalysts, movers, risks)
- Time-stamped with data freshness
```

## Input Format (JSON)

```json
{
  "date": "2026-01-20",
  "timezone": "America/New_York",
  "market_context": {
    "indices": {
      "SPY": { "price": 498.50, "change_pct": 0.45 },
      "QQQ": { "price": 425.20, "change_pct": 0.72 },
      "VIX": 14.5
    },
    "session": "pre_market"
  },
  "watchlist": [
    {
      "ticker": "AAPL",
      "snapshot": { "price": 197.45, "change_pct": 1.2, "volume_ratio": 0.8 },
      "catalysts": [{ "type": "earnings", "date": "2026-01-30" }],
      "top_news": ["Supply chain report positive"],
      "alerts_triggered_last_24h": 1
    }
  ]
}
```

## Output Requirements

Return:

1. **JSON** conforming to `schemas/daily-briefing.schema.json`
2. 2. **Mobile-friendly Markdown** (<= 2000 characters)
  
   3. ## Sections (in order)
  
   4. ### 1. Today's Catalysts
   5. - Earnings today or this week
      - - Ex-dividend dates
        - - Known events (FDA, conferences)
         
          - ### 2. Biggest Movers
          - - Top gainers/losers from watchlist (pre-market or prior close)
            - - Include % change and brief reason if known
             
              - ### 3. Notable Headlines
              - - 1-3 headlines from major sources only
                - - Focus on watchlist-relevant news
                 
                  - ### 4. Risk Notes
                  - - Earnings proximity warnings
                    - - Unusual volatility flags
                      - - Low liquidity warnings
                       
                        - ### 5. Market Context (brief)
                        - - Index direction (SPY/QQQ)
                          - - VIX level context
                           
                            - ## Rules
                           
                            - - **No advice language**: avoid "buy", "sell", "should", "recommend"
                              - - **Prefer clarity over completeness**: skip sections if no relevant data
                                - - **Keep it scannable**: headers + bullets, minimal prose
                                  - - **Include "data as of" timestamps**
                                    - - **Max 5 tickers featured** (prioritize by catalyst proximity or move size)
                                     
                                      - ## Example Output (Markdown)
                                     
                                      - ```markdown
                                        # Daily Briefing — Jan 20, 2026
                                        *Data as of 8:45 AM ET*

                                        ## 📅 Today's Catalysts
                                        - **NFLX** earnings after close
                                        - **MSFT** ex-dividend date

                                        ## 📈 Watchlist Movers (Pre-market)
                                        | Ticker | Change | Note |
                                        |--------|--------|------|
                                        | AAPL | +1.8% | Supply chain optimism |
                                        | NVDA | +2.1% | AI demand reports |
                                        | META | -0.5% | No specific catalyst |

                                        ## 📰 Headlines
                                        - AAPL: Supplier reports strong orders (Reuters)
                                        - NVDA: Data center revenue beat estimates (Bloomberg)

                                        ## ⚠️ Risk Notes
                                        - **NFLX**: Earnings today — expect elevated IV
                                        - **TSLA**: 3 alerts in 24h — high volatility regime

                                        ## 🌍 Market Context
                                        SPY +0.5% pre-market | VIX 14.5 (low)

                                        ---
                                        *Informational only. Not financial advice.*
                                        ```

                                        ## n8n Integration Notes

                                        - Trigger: Cron at 8:30 AM ET (or user-defined)
                                        - - Requires: Watchlist from Notion, quote data, news summaries
                                          - - Output: Discord/Telegram message + store in Notion Daily Briefings table
                                            - - Optional: Include link to full briefing page in Notion
