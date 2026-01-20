# MVP Scope

## Phase 1: Foundation & Specifications

### What's In Scope

**Infrastructure**
- GitHub repository with organized folder structure
- - JSON schemas for all data contracts (Intel Card, Alert Event, Watchlist Item, Daily Briefing)
  - - YAML configuration files for thresholds, cooldowns, and filters
    - - LLM prompt templates for AI-generated content
      - - n8n workflow specifications (not full implementation)
       
        - **Notion Databases**
        - - Watchlist database (ticker tracking)
          - - Intel Cards database (generated research storage)
            - - Journal database (alert history and cooldown tracking)
             
              - **Workflow Specifications**
              - - Market Pulse (scheduled alerting workflow)
                - - Intel Card Generator (on-demand research workflow)
                  - - Alert delivery to Discord/Telegram/Email
                   
                    - ### What's NOT In Scope (Phase 1)
                   
                    - - Full n8n workflow implementation
                      - - Data provider integration
                        - - Mobile app
                          - - Broker API integration
                            - - Real-time websocket streaming
                              - - Backtesting or historical analysis
                                - - Multi-user support
                                  - - Authentication/authorization
                                   
                                    - ### Deliverables Checklist
                                   
                                    - #### Repository Structure
                                    - - [x] README.md with Phase 1 overview
                                      - [ ] - [x] .gitignore
                                      - [ ] - [x] LICENSE (MIT)
                                     
                                      - [ ] #### Schemas (/schemas)
                                      - [ ] - [x] intel-card.schema.json
                                      - [ ] - [x] alert-event.schema.json
                                      - [ ] - [x] watchlist-item.schema.json
                                      - [ ] - [x] daily-briefing.schema.json
                                     
                                      - [ ] #### Configurations (/configs)
                                      - [ ] - [x] alert-thresholds.yml
                                      - [ ] - [x] cooldowns.yml
                                      - [ ] - [x] noise-filters.yml
                                      - [ ] - [ ] scoring-weights.yml (optional)
                                     
                                      - [ ] #### Prompts (/prompts)
                                      - [ ] - [x] intel-card.prompt.md
                                      - [ ] - [x] risk-flags.prompt.md
                                      - [ ] - [ ] daily-briefing.prompt.md
                                      - [ ] - [ ] what-changed.prompt.md
                                     
                                      - [ ] #### Workflows (/workflows)
                                      - [ ] - [x] market-pulse.workflow.md
                                      - [ ] - [x] ticker-intel-card.workflow.md
                                     
                                      - [ ] #### Notion (/notion)
                                      - [ ] - [x] database-schemas.md
                                     
                                      - [ ] #### Documentation (/docs)
                                      - [ ] - [x] 01-vision.md
                                      - [ ] - [x] 02-mvp-scope.md (this file)
                                      - [ ] - [ ] 03-user-stories.md
                                      - [ ] - [ ] 04-architecture.md
                                      - [ ] - [ ] 05-alert-rules.md
                                      - [ ] - [ ] 06-data-sources.md
                                      - [ ] - [ ] 07-security-keys.md
                                      - [ ] - [ ] 08-non-goals.md
                                     
                                      - [ ] ### Success Criteria
                                     
                                      - [ ] Phase 1 is complete when:
                                     
                                      - [ ] 1. All checked items above are committed to the repository
                                      - [ ] 2. Schemas validate against JSON Schema spec
                                      - [ ] 3. Notion databases can be created from the schema documentation
                                      - [ ] 4. Workflow specs are detailed enough to implement in n8n
                                      - [ ] 5. A developer can understand the system from documentation alone
                                     
                                      - [ ] ### Next Phase Preview
                                     
                                      - [ ] Phase 2 will focus on:
                                      - [ ] - Implementing n8n workflows with real data providers
                                      - [ ] - Setting up Discord/Telegram bots for alert delivery
                                      - [ ] - Creating Notion databases and integrations
                                      - [ ] - Tuning thresholds based on real alert volume
