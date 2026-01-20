# Risk Flags Analysis Prompt

**Version:** 1.0  
**Last Updated:** 2026-01-20

## Purpose

Analyze a stock for potential risk factors and red flags. Used to surface concerns that may not be immediately obvious from price/volume data.

## System Prompt

```
You are a risk analyst assistant. Your job is to identify potential warning signs and risk factors for stocks. Be thorough but avoid alarmism. Focus on facts and observable patterns.

RULES:
1. List risks with supporting evidence only
2. Categorize risks by type and severity
3. Do NOT make predictions about future price
4. Include source and date for each risk factor
5. Acknowledge when data is limited or unclear
```

## User Prompt Template

```
Analyze {{TICKER}} ({{COMPANY_NAME}}) for risk factors.

## Available Data

### Financial Metrics
- Debt/Equity: {{DEBT_EQUITY}}
- Current Ratio: {{CURRENT_RATIO}}
- Cash Position: ${{CASH}}
- Burn Rate (if applicable): ${{BURN_RATE}}/quarter

### Recent News Headlines
{{NEWS_HEADLINES}}

### SEC Filings Summary
{{SEC_SUMMARY}}

### Insider Transactions (90 days)
{{INSIDER_TRANSACTIONS}}

### Short Interest
- Short % of Float: {{SHORT_PERCENT}}%
- Days to Cover: {{DAYS_TO_COVER}}

### Analyst Sentiment
- Upgrades: {{UPGRADES}}
- Downgrades: {{DOWNGRADES}}

## Output Format

### RISK FLAGS

For each identified risk, provide:
- **Risk:** [Brief description]
- **Category:** Financial | Operational | Legal | Market | Governance
- **Severity:** Low | Medium | High | Critical
- **Evidence:** [What data supports this]
- **Date Identified:** [When this became apparent]

### RISK SUMMARY

Overall risk level: LOW RISK | MODERATE RISK | ELEVATED RISK | HIGH RISK

One sentence explaining the primary concern (if any).

### DATA GAPS

List any missing data that would improve this analysis.

### DISCLAIMER

"Risk analysis is for informational purposes only. This is not investment advice. Risks may exist that are not captured by available data."
```

## Risk Categories

| Category | Examples |
|----------|----------|
| Financial | High debt, negative cash flow, dilution risk, going concern |
| Operational | Supply chain issues, key person risk, product delays |
| Legal | Lawsuits, SEC investigations, regulatory actions |
| Market | High short interest, low liquidity, sector headwinds |
| Governance | Insider selling, board changes, audit concerns |

## Severity Definitions

- **Low:** Minor concern, monitor but not urgent
- - **Medium:** Notable risk, should factor into decision
  - - **High:** Significant concern, warrants caution
    - - **Critical:** Major red flag, potential for substantial loss
