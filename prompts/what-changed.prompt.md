# What Changed Prompt

**Version:** 1.0  
**Last Updated:** 2026-01-20

## Purpose

Summarize what changed for a ticker since the previous Intel Card. Used for quick delta reviews without regenerating a full card.

## System Prompt

```
You summarize *what changed* for a ticker since the previous Intel Card. Informational only — not financial advice.
```

## Input Format (JSON)

```json
{
  "ticker": "AAPL",
  "current_card": { /* latest Intel Card fields */ },
  "previous_card": { /* prior Intel Card fields; may be null */ },
  "max_bullets": 6
}
```

## Output Requirements

Return:

1. **JSON** conforming to this structure:
2. ```json
   {
     "ticker": "AAPL",
     "as_of": "2026-01-20T09:30:00-05:00",
     "previous_as_of": "2026-01-15T16:00:00-05:00",
     "changes": [
       {
         "category": "price|volume|news|filing|catalyst|risk",
         "change": "Brief description of what changed",
         "impact": "Why it matters (1 clause)"
       }
     ],
     "confidence": "high|medium|low",
     "disclaimer": "Informational only. Not financial advice."
   }
   ```

   2. **Mobile-friendly Markdown** (<= 1200 characters)
  
   3. ## Rules
  
   4. - If `previous_card` is null/missing: output "No prior snapshot — here's what stands out now." and list current highlights instead of deltas
      - - Focus on **concrete deltas**:
        -   - Price change (absolute and %)
            -   - Volume regime change (normal → elevated)
                -   - New headlines since last card
                    -   - New SEC filings
                        -   - Catalyst date changes (earnings moved, ex-div added)
                            -   - Risk score or flags changed
                                - - **Avoid advice words**: "buy", "sell", "should", "guarantee", "recommend"
                                  - - Always include timestamps for both current and previous snapshots
                                    - - Limit to `max_bullets` changes (default 6)
                                      - - Prioritize by significance, not recency
                                       
                                        - ## Change Bullet Format
                                       
                                        - Each bullet should follow this pattern:
                                        - > **[Category]**: Change description → Why it matters
                                          >
                                          > Examples:
                                          > - **Risk**: New flag added (earnings within 7 days) → Volatility risk elevated around event
                                          > - - **Filing**: New 8-K filed (Jan 18) → Review for material updates
                                          >   - - **Price**: Up 8.5% since last card → Now trading above 50-day MA
                                          >     - - **Catalyst**: Earnings moved from Feb 1 to Jan 28 → Sooner than expected, prepare thesis
                                          >      
                                          >       - ## Confidence Levels
                                          >      
                                          >       - - **High**: Both cards have complete data, clear deltas identified
                                          >         - - **Medium**: Some data missing or stale, deltas are best-effort
                                          >           - - **Low**: Previous card very old (>30 days) or major data gaps
                                          >            
                                          >             - ## Example Output (Markdown)
                                          >            
                                          >             - ```markdown
                                          >               ## AAPL — What Changed
                                          >               *Current: Jan 20, 2026 09:30 ET | Previous: Jan 15, 2026 16:00 ET*
                                          >
                                          >               **Price**: +4.2% ($189.50 → $197.45) → Broke above resistance at $195
                                          >
                                          >               **Volume**: 2.1x average today → Unusual activity, monitor for catalyst
                                          >
                                          >               **Filing**: Form 4 filed Jan 17 (CFO sold 10K shares) → Routine sale per 10b5-1 plan
                                          >
                                          >               **Catalyst**: Earnings confirmed Jan 30 → 10 days away, expect IV expansion
                                          >
                                          >               **Risk**: Risk score unchanged (42/100) → No new flags
                                          >
                                          >               ---
                                          >               *Informational only. Not financial advice.*
                                          >               ```
                                          >
                                          > ## n8n Integration Notes
                                          >
                                          > - Call this prompt when user requests `/delta TICKER` or when comparing cards
                                          > - - Requires both current and previous Intel Card data
                                          >   - - Can be triggered automatically when generating a new Intel Card to show delta from last
