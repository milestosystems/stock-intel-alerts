# Stock Market Intel + Alerts System

**Phase 1: Foundation & Specifications**

A mobile-first stock market intelligence and alerting system designed to provide actionable, low-noise market updates. Built for use alongside Webull (execution platform) with no trading integration required.

> ⚠️ **DISCLAIMER**: This system provides market information for educational and research purposes only. It does not constitute financial advice. Always conduct your own research and consult qualified financial advisors before making investment decisions. Data shown may be delayed or inaccurate.
>
> ## 🎯 Purpose
>
> Push relevant market updates and generate comprehensive "Intel Cards" for any US equity ticker you're researching, with emphasis on:
>
> - **Investable signals** over noise
> - - **Risk awareness** and red flags
>   - - **Timestamp transparency** on all data
>     - - **Mobile-first delivery** (Discord/Telegram/Email)
>      
>       - ## 📦 What's in This Repository
>      
>       - This is the **Phase 1 Foundation** — specifications and schemas that enable immediate implementation with n8n + Notion.
>      
>       - | Folder | Contents |
> |--------|----------|
> | `/docs` | Vision, MVP scope, user stories, architecture, alert rules |
> | `/schemas` | JSON Schemas for watchlist, intel cards, alerts, briefings |
> | `/prompts` | LLM prompt templates with versioning |
> | `/configs` | YAML configs for thresholds, cooldowns, scoring |
> | `/workflows` | n8n workflow specifications (pseudo-code + node specs) |
> | `/notion` | Notion database schema recommendations |
>
> ## 🚀 Quick Start (Phase 1)
>
> ### Prerequisites
>
> - n8n instance (self-hosted or cloud)
> - - Notion workspace
>   - - Discord/Telegram bot (for alerts)
>     - - API keys for data providers (see `/docs/06-data-sources.md`)
>      
>       - ### Setup Steps
>      
>       - 1. **Clone this repository**
>         2.    ```bash
>                  git clone https://github.com/milestosystems/stock-intel-alerts.git
>                  cd stock-intel-alerts
>                  ```
>
> 2. **Review the documentation**
> 3.    - Start with `/docs/01-vision.md` for system overview
>       -    - Check `/docs/02-mvp-scope.md` for Phase 1 boundaries
>        
>            - 3. **Set up Notion databases**
>              4.    - Follow schemas in `/notion/database-schemas.md`
>                    -    - Create: Watchlist, Intel Cards, Journal
>                     
>                         - 4. **Configure n8n workflows**
>                           5.    - Use `/workflows/*.workflow.md` as implementation guides
>                                 -    - Import placeholder workflows from n8n community templates
>                                  
>                                      - 5. **Set environment variables**
>                                        6.    - Reference `/docs/07-security-keys.md` for required API keys
>                                              -    - Never commit keys to the repository
>                                               
>                                                   - ## 📊 Core Features (Phase 1 MVP)
>                                               
>                                                   - ### Intel Card
>                                                   - On-demand "background check" for any ticker:
>                                                   - - Price snapshot with trend context
> - Upcoming catalysts (earnings, ex-div dates)
> - - Recent headlines (3-5 items)
>   - - Latest SEC filings summary
>     - - Risk flags (volatility, news sentiment)
>       - - AI-generated synthesis
>        
>         - ### Alerts (Rule-Based)
>         - - Significant price moves (>3% intraday)
>           - - Earnings reminders (T-7, T-1)
>             - - Unusual volume spikes (>2x avg)
>               - - New SEC filings (10-K, 10-Q, 8-K)
>                 - - Breaking news mentions
>                  
>                   - ### Daily Briefing
>                   - Morning summary for watchlist:
>                   - - Pre-market movers
>                     - - Today's earnings/events
>                       - - Overnight news highlights
>                         - - Risk items to watch
>                          
>                           - ## 🛡️ Design Principles
>                          
>                           - 1. **Low Noise**: Cooldowns prevent alert spam; severity levels filter triviality
>                             2. 2. **Transparency**: Every data point includes timestamp and source
>                                3. 3. **Explainability**: Every alert includes "Why am I seeing this?"
>                                   4. 4. **No Financial Advice**: Informational only; no "buy/sell" language
>                                      5. 5. **Provider Agnostic**: Swap data sources without changing schemas
>                                        
>                                         6. ## 📁 Key Files Reference
>                                        
>                                         7. | File | Description |
>                                         8. |------|-------------|
>                                         9. | `schemas/intel-card.schema.json` | Complete Intel Card data contract |
> | `schemas/alert-event.schema.json` | Alert payload structure |
> | `configs/alert-thresholds.yml` | Configurable trigger thresholds |
> | `configs/cooldowns.yml` | Alert deduplication settings |
> | `prompts/intel-card.prompt.md` | LLM prompt for Intel Card synthesis |
>
> ## 🔮 Phase 2 Preview
>
> Future phases may include:
> - Sector/index correlation analysis
> - - Options flow alerts
>   - - Insider trading pattern detection
>     - - Portfolio exposure tracking
>       - - Mobile app (React Native)
>        
>         - See `/docs/08-non-goals.md` for explicit boundaries.
>        
>         - ## 📄 License
>        
>         - MIT License - See LICENSE file
>
> ## ⚠️ Important Disclaimers
>
> - **Not Financial Advice**: This tool provides information only. It does not recommend securities or trading strategies.
> - - **Data Accuracy**: Market data may be delayed 15+ minutes. Verify with official sources before decisions.
>   - - **No Guarantees**: Past performance and patterns do not predict future results.
>     - - **Personal Responsibility**: Users are solely responsible for their investment decisions.
>      
>       - ---
>
> *Built with ❤️ for informed market awareness*
