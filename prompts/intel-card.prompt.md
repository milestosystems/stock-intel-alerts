# Intel Card Generation Prompt

**Version:** 1.0  
**Last Updated:** 2026-01-20  
**Model:** Claude 3.5 Sonnet / GPT-4o (configurable)

## Purpose

Generate a comprehensive yet concise Intel Card for a stock ticker. The Intel Card provides a snapshot of the current state of a company, synthesizing multiple data sources into actionable intelligence.

## System Prompt

```
You are a financial research analyst assistant. Your role is to synthesize market data, news, and filings into clear, actionable intelligence briefs.

CRITICAL RULES:
1. NEVER provide buy/sell/hold recommendations
2. ALWAYS include data timestamps and sources
3. Flag uncertainty explicitly - say "unclear" or "conflicting signals" when appropriate
4. Keep language neutral and factual
5. Prioritize recent information over historical
6. Highlight what has CHANGED recently
7. End every response with the disclaimer

OUTPUT FORMAT: Structure your response exactly as specified in the user prompt.
```

## User Prompt Template

```
Generate an Intel Card for {{TICKER}} ({{COMPANY_NAME}}).

## Input Data

### Current Quote
- Price: ${{CURRENT_PRICE}}
- Change: {{PRICE_CHANGE}} ({{PRICE_CHANGE_PCT}}%)
- Volume: {{VOLUME}} ({{VOLUME_VS_AVG}}x average)
- Market Cap: ${{MARKET_CAP}}
- 52-Week Range: ${{WEEK_52_LOW}} - ${{WEEK_52_HIGH}}

### Recent News (last 7 days)
{{NEWS_ITEMS}}

### Recent SEC Filings (last 30 days)
{{SEC_FILINGS}}

### Analyst Consensus
- Rating: {{ANALYST_RATING}}
- Price Target: ${{PRICE_TARGET_AVG}} (Low: ${{PT_LOW}}, High: ${{PT_HIGH}})
- Recent Changes: {{ANALYST_CHANGES}}

### Key Metrics
- P/E Ratio: {{PE_RATIO}}
- EPS (TTM): ${{EPS}}
- Revenue (TTM): ${{REVENUE}}
- Next Earnings: {{EARNINGS_DATE}}

### Insider Activity (last 90 days)
{{INSIDER_ACTIVITY}}

## Output Requirements

Generate an Intel Card with these sections:

### 1. HEADLINE (1 line)
A single sentence capturing the current situation. Start with the ticker.

### 2. CURRENT STATE (2-3 sentences)
What is happening RIGHT NOW with this stock? Focus on price action, volume, and immediate catalysts.

### 3. WHAT CHANGED (bullet points)
List 3-5 recent changes or developments. Each bullet should include:
- What changed
- When it changed
- Why it matters

### 4. KEY RISKS (bullet points)
List 2-4 risk factors to watch. Be specific and actionable.

### 5. UPCOMING CATALYSTS (bullet points)
List known upcoming events that could move the stock:
- Earnings dates
- FDA decisions
- Product launches
- Conference presentations

### 6. SIGNAL SUMMARY
One of: BULLISH SIGNALS | BEARISH SIGNALS | MIXED SIGNALS | NEUTRAL
With a 1-sentence explanation of why.

### 7. DATA FRESHNESS
List the timestamp of each data source used.

### 8. DISCLAIMER
"This Intel Card is for informational purposes only and does not constitute financial advice. Always conduct your own research and consult qualified professionals before making investment decisions. Data may be delayed or incomplete."
```

## Example Output

```
## AAPL Intel Card
Generated: 2026-01-20 09:45 EST

### HEADLINE
AAPL trading near all-time highs on strong iPhone demand, with earnings approaching in 2 weeks.

### CURRENT STATE
Apple is up 2.3% today on above-average volume following positive supply chain reports. The stock is trading at $198.50, just 3% below its 52-week high of $204.12. Momentum has been building since the December guidance raise.

### WHAT CHANGED
- **iPhone shipments up 15%** (Jan 15) - Supply chain data suggests stronger-than-expected holiday sales; bullish for upcoming earnings
- **Services revenue record** (Jan 10) - App Store spending hit new highs in December; supports premium valuation
- **China concerns easing** (Jan 8) - Government restrictions on iPhone use appear limited in scope; removes overhang
- **AI features announced** (Jan 5) - New Siri capabilities coming in iOS 19; potential growth driver

### KEY RISKS
- **Valuation stretched**: P/E of 32x is above 5-year average of 25x
- **China exposure**: 18% of revenue from region with regulatory uncertainty
- **Services growth slowing**: Decelerated from 24% to 16% YoY
- **Antitrust scrutiny**: DOJ lawsuit ongoing with trial in Q3

### UPCOMING CATALYSTS
- **Earnings**: Jan 30, 2026 (after market close)
- **WWDC**: June 2026 (AI strategy reveal expected)
- **iPhone 17 launch**: September 2026

### SIGNAL SUMMARY
BULLISH SIGNALS - Strong price momentum, positive analyst sentiment, and approaching earnings with raised guidance create favorable near-term setup. Watch for execution on AI narrative.

### DATA FRESHNESS
- Quote: Real-time (09:45 EST)
- News: Last 7 days (Jan 13-20)
- SEC Filings: Last filing Dec 15
- Analyst Data: Updated Jan 18

### DISCLAIMER
This Intel Card is for informational purposes only and does not constitute financial advice. Always conduct your own research and consult qualified professionals before making investment decisions. Data may be delayed or incomplete.
```

## Variable Placeholders

| Variable | Description | Source |
|----------|-------------|--------|
| {{TICKER}} | Stock symbol | User input |
| {{COMPANY_NAME}} | Full company name | Data provider |
| {{CURRENT_PRICE}} | Current stock price | Quote API |
| {{PRICE_CHANGE}} | Dollar change | Quote API |
| {{PRICE_CHANGE_PCT}} | Percent change | Quote API |
| {{VOLUME}} | Today's volume | Quote API |
| {{VOLUME_VS_AVG}} | Volume ratio | Calculated |
| {{MARKET_CAP}} | Market capitalization | Quote API |
| {{WEEK_52_LOW}} | 52-week low | Quote API |
| {{WEEK_52_HIGH}} | 52-week high | Quote API |
| {{NEWS_ITEMS}} | Recent news summaries | News API |
| {{SEC_FILINGS}} | Recent filings | SEC EDGAR |
| {{ANALYST_RATING}} | Consensus rating | Data provider |
| {{PRICE_TARGET_AVG}} | Average price target | Data provider |
| {{PE_RATIO}} | Price-to-earnings | Calculated |
| {{EPS}} | Earnings per share | Data provider |
| {{REVENUE}} | Trailing revenue | Data provider |
| {{EARNINGS_DATE}} | Next earnings | Calendar API |
| {{INSIDER_ACTIVITY}} | Insider transactions | SEC Form 4 |

## Notes

- Keep Intel Cards under 500 words for mobile readability
- - Use markdown formatting for structure
  - - Bold key terms and numbers for scannability
    - - Include "N/A" or "No recent data" when information is missing
      - - Timestamp all generated cards
