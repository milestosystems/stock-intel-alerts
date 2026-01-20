# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.1.0] - 2026-01-20

### Added

**Repository Structure**
- Initial repository setup with README, LICENSE (MIT), and .gitignore

- **Schemas (/schemas)**
- - `intel-card.schema.json` - Data contract for on-demand ticker research
  - - `alert-event.schema.json` - Data contract for market alerts
    - - `watchlist-item.schema.json` - Data contract for watchlist entries
      - - `daily-briefing.schema.json` - Data contract for daily market summaries
       
        - **Configurations (/configs)**
        - - `alert-thresholds.yml` - Price, volume, and event thresholds
          - - `cooldowns.yml` - Alert cooldown and rate limiting rules
            - - `noise-filters.yml` - Filters to reduce alert noise
             
              - **Prompts (/prompts)**
              - - `intel-card.prompt.md` - LLM prompt for Intel Card generation
                - - `risk-flags.prompt.md` - LLM prompt for risk analysis
                 
                  - **Workflows (/workflows)**
                  - - `market-pulse.workflow.md` - n8n workflow spec for scheduled alerts
                    - - `ticker-intel-card.workflow.md` - n8n workflow spec for on-demand research
                     
                      - **Notion (/notion)**
                      - - `database-schemas.md` - Notion database structure for Watchlist, Intel Cards, and Journal
                       
                        - **Documentation (/docs)**
                        - - `01-vision.md` - Project vision and design principles
                          - - `02-mvp-scope.md` - Phase 1 scope and deliverables checklist
                           
                            - ### Phase 1 Status
                           
                            - Phase 1 foundation is substantially complete. Remaining items tracked in `02-mvp-scope.md`.
                           
                            - ---

                            ## [Unreleased]

                            ### Planned for Phase 2
                            - n8n workflow implementation
                            - - Data provider integration
                              - - Discord/Telegram bot setup
                                - - Notion database creation
                                  - - Threshold tuning based on real alerts
