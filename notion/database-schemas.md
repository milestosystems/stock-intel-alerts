# Notion Database Schemas

This document defines the three core Notion databases for the Stock Intel + Alerts system.

## Overview

| Database | Purpose | Update Frequency |
|----------|---------|------------------|
| Watchlist | Tickers to monitor | User-managed |
| Intel Cards | Generated analysis snapshots | On-demand + scheduled |
| Journal | Alert history and notes | Automatic + user notes |

---

## Database 1: Watchlist

**Purpose**: Track tickers you want to monitor for alerts and include in daily briefings.

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| Ticker | Title | Yes | Stock symbol (e.g., AAPL, TSLA) |
| Company Name | Text | No | Full company name (auto-populated) |
| Added Date | Date | Yes | When ticker was added to watchlist |
| Status | Select | Yes | Active, Paused, Archived |
| Thesis | Text | No | Your investment thesis or notes |
| Target Price | Number | No | Your target price (optional) |
| Stop Loss | Number | No | Your stop loss level (optional) |
| Position Size | Number | No | Number of shares held (optional) |
| Sector | Select | No | Industry sector |
| Alert Priority | Select | Yes | High, Medium, Low |
| Last Intel Card | Date | No | When last Intel Card was generated |
| Tags | Multi-select | No | Custom tags for filtering |

### Select Options

**Status**:
- `Active` - Receiving alerts and in briefings
- - `Paused` - Temporarily not receiving alerts
  - - `Archived` - Removed from active monitoring
   
    - **Alert Priority**:
    - - `High` - All alerts, immediate delivery
      - - `Medium` - Standard alerts, batched delivery OK
        - - `Low` - Only major alerts (earnings, filings)
         
          - **Sector** (examples):
          - - Technology
            - - Healthcare
              - - Financial
                - - Consumer
                  - - Energy
                    - - Industrial
                      - - Real Estate
                        - - Utilities
                          - - Materials
                            - - Communication
                             
                              - ### Views
                             
                              - 1. **Active Watchlist** - Filter: Status = Active, Sort: Ticker A-Z
                                2. 2. **By Sector** - Group by Sector
                                   3. 3. **High Priority** - Filter: Alert Priority = High
                                      4. 4. **Recently Added** - Sort: Added Date descending
                                         5. 5. **Needs Review** - Filter: Last Intel Card > 7 days ago
                                           
                                            6. ### Workflow Integration
                                           
                                            7. **n8n reads from this database to**:
                                            8. - Get list of tickers for Market Pulse scans
                                               - - Determine alert priority routing
                                                 - - Filter by status (skip Paused/Archived)
                                                  
                                                   - **n8n writes to this database to**:
                                                   - - Update Last Intel Card date
                                                     - - Auto-populate Company Name on new entries
                                                      
                                                       - ---

                                                       ## Database 2: Intel Cards

                                                       **Purpose**: Store generated Intel Card snapshots for historical reference and comparison.

                                                       ### Properties

                                                       | Property | Type | Required | Description |
                                                       |----------|------|----------|-------------|
                                                       | Title | Title | Yes | "{Ticker} - {Date}" format |
                                                       | Ticker | Text | Yes | Stock symbol |
                                                       | Generated At | Date | Yes | Timestamp of generation |
                                                       | Trigger | Select | Yes | How the card was generated |
                                                       | Current Price | Number | Yes | Price at time of generation |
                                                       | Day Change % | Number | Yes | Intraday change percentage |
                                                       | Market Cap | Number | No | Market capitalization |
                                                       | PE Ratio | Number | No | Price to earnings ratio |
                                                       | 52W High | Number | No | 52-week high price |
                                                       | 52W Low | Number | No | 52-week low price |
                                                       | Next Earnings | Date | No | Upcoming earnings date |
                                                       | Risk Score | Number | Yes | Calculated risk score (1-10) |
                                                       | Risk Flags | Multi-select | No | Identified risk factors |
                                                       | Headlines JSON | Text | No | JSON array of recent headlines |
                                                       | Filings JSON | Text | No | JSON array of recent SEC filings |
                                                       | AI Summary | Text | Yes | AI-generated synthesis |
                                                       | Raw Data JSON | Text | No | Complete raw data for debugging |
                                                       | Related Watchlist | Relation | No | Link to Watchlist entry |

                                                       ### Select Options

                                                       **Trigger**:
                                                       - `On-Demand` - User requested via command
                                                       - - `Scheduled` - Daily/weekly automatic refresh
                                                         - - `Alert-Triggered` - Generated due to significant event
                                                          
                                                           - **Risk Flags** (examples):
                                                           - - High Volatility
                                                             - - Negative Sentiment
                                                               - - Earnings Imminent
                                                                 - - Recent SEC Filing
                                                                   - - Price Near 52W Low
                                                                     - - Price Near 52W High
                                                                       - - Volume Spike
                                                                         - - Insider Selling
                                                                          
                                                                           - ### Views
                                                                          
                                                                           - 1. **Recent Cards** - Sort: Generated At descending
                                                                             2. 2. **By Ticker** - Group by Ticker
                                                                                3. 3. **High Risk** - Filter: Risk Score >= 7
                                                                                   4. 4. **This Week** - Filter: Generated At is within past week
                                                                                     
                                                                                      5. ### Workflow Integration
                                                                                     
                                                                                      6. **n8n writes to this database**:
                                                                                      7. - Creates new entry for each Intel Card generated
                                                                                         - - Stores all data fields from aggregation
                                                                                           - - Links to Watchlist entry if ticker is watched
                                                                                            
                                                                                             - **User can**:
                                                                                             - - Add personal notes to any Intel Card
                                                                                               - - Compare cards over time for same ticker
                                                                                                
                                                                                                 - ---

                                                                                                 ## Database 3: Journal

                                                                                                 **Purpose**: Log all alerts sent and allow user annotations.

                                                                                                 ### Properties

                                                                                                 | Property | Type | Required | Description |
                                                                                                 |----------|------|----------|-------------|
                                                                                                 | Title | Title | Yes | Alert summary line |
                                                                                                 | Timestamp | Date | Yes | When alert was sent |
                                                                                                 | Ticker | Text | Yes | Related stock symbol |
                                                                                                 | Alert Type | Select | Yes | Type of alert |
                                                                                                 | Severity | Select | Yes | Alert severity level |
                                                                                                 | Trigger Value | Text | No | The value that triggered alert |
                                                                                                 | Threshold | Text | No | The threshold that was exceeded |
                                                                                                 | Message | Text | Yes | Full alert message sent |
                                                                                                 | Channel | Select | Yes | Where alert was delivered |
                                                                                                 | Acknowledged | Checkbox | No | User marked as reviewed |
                                                                                                 | User Notes | Text | No | User's personal notes |
                                                                                                 | Action Taken | Select | No | What user did in response |
                                                                                                 | Related Intel Card | Relation | No | Link to Intel Card if generated |
                                                                                                 | Related Watchlist | Relation | No | Link to Watchlist entry |
                                                                                                 | Cooldown Key | Text | No | Key used for deduplication |

                                                                                                 ### Select Options

                                                                                                 **Alert Type**:
                                                                                                 - `Price Move` - Significant price change
                                                                                                 - - `Volume Spike` - Unusual volume
                                                                                                   - - `Gap` - Opening gap from previous close
                                                                                                     - - `Volatility` - Elevated price range
                                                                                                       - - `Earnings Reminder` - Upcoming earnings
                                                                                                         - - `SEC Filing` - New SEC document
                                                                                                           - - `News Headline` - Major news mention
                                                                                                             - - `Daily Briefing` - Morning summary
                                                                                                              
                                                                                                               - **Severity**:
                                                                                                               - - `Critical` - Immediate attention needed
                                                                                                                 - - `High` - Important, review soon
                                                                                                                   - - `Medium` - Noteworthy
                                                                                                                     - - `Low` - Informational
                                                                                                                      
                                                                                                                       - **Channel**:
                                                                                                                       - - `Discord`
                                                                                                                         - - `Telegram`
                                                                                                                           - - `Email`
                                                                                                                             - - `All`
                                                                                                                              
                                                                                                                               - **Action Taken** (user-selected):
                                                                                                                               - - `No Action` - Reviewed, no action needed
                                                                                                                                 - - `Bought More` - Increased position
                                                                                                                                   - - `Sold Some` - Reduced position
                                                                                                                                     - - `Sold All` - Exited position
                                                                                                                                       - - `Set Alert` - Created manual alert
                                                                                                                                         - - `Researched` - Did additional research
                                                                                                                                          
                                                                                                                                           - ### Views
                                                                                                                                          
                                                                                                                                           - 1. **Today's Alerts** - Filter: Timestamp is today
                                                                                                                                             2. 2. **Unacknowledged** - Filter: Acknowledged = false
                                                                                                                                                3. 3. **By Ticker** - Group by Ticker
                                                                                                                                                   4. 4. **Critical Alerts** - Filter: Severity = Critical
                                                                                                                                                      5. 5. **This Week** - Filter: Timestamp is within past week
                                                                                                                                                        
                                                                                                                                                         6. ### Workflow Integration
                                                                                                                                                        
                                                                                                                                                         7. **n8n writes to this database**:
                                                                                                                                                         8. - Creates entry for every alert sent
                                                                                                                                                            - - Populates all alert metadata
                                                                                                                                                              - - Sets cooldown key for deduplication checks
                                                                                                                                                               
                                                                                                                                                                - **n8n reads from this database**:
                                                                                                                                                                - - Checks cooldown keys to prevent duplicate alerts
                                                                                                                                                                  - - Gets timestamp of last alert for each ticker/type
                                                                                                                                                                   
                                                                                                                                                                    - ---
                                                                                                                                                                    
                                                                                                                                                                    ## Setup Instructions
                                                                                                                                                                    
                                                                                                                                                                    ### Step 1: Create Databases
                                                                                                                                                                    
                                                                                                                                                                    1. In Notion, create a new page called "Stock Intel Hub"
                                                                                                                                                                    2. 2. Create three inline databases: Watchlist, Intel Cards, Journal
                                                                                                                                                                       3. 3. Add all properties as specified above
                                                                                                                                                                         
                                                                                                                                                                          4. ### Step 2: Create Notion Integration
                                                                                                                                                                         
                                                                                                                                                                          5. 1. Go to https://www.notion.so/my-integrations
                                                                                                                                                                             2. 2. Create new integration called "Stock Intel Alerts"
                                                                                                                                                                                3. 3. Copy the Internal Integration Token
                                                                                                                                                                                   4. 4. Store token securely (never in code)
                                                                                                                                                                                     
                                                                                                                                                                                      5. ### Step 3: Share Databases with Integration
                                                                                                                                                                                     
                                                                                                                                                                                      6. 1. Open each database
                                                                                                                                                                                         2. 2. Click "..." menu → "Add connections"
                                                                                                                                                                                            3. 3. Select your "Stock Intel Alerts" integration
                                                                                                                                                                                               4. 4. Repeat for all three databases
                                                                                                                                                                                                 
                                                                                                                                                                                                  5. ### Step 4: Get Database IDs
                                                                                                                                                                                                 
                                                                                                                                                                                                  6. 1. Open each database as a full page
                                                                                                                                                                                                     2. 2. Copy the URL
                                                                                                                                                                                                        3. 3. Extract the database ID (32-character string after workspace name)
                                                                                                                                                                                                           4. 4. Example: `https://notion.so/workspace/abc123...` → ID is `abc123...`
                                                                                                                                                                                                             
                                                                                                                                                                                                              5. ### Step 5: Configure n8n
                                                                                                                                                                                                             
                                                                                                                                                                                                              6. Store these as n8n credentials:
                                                                                                                                                                                                              7. - `NOTION_API_KEY` - Integration token
                                                                                                                                                                                                                 - - `NOTION_WATCHLIST_DB` - Watchlist database ID
                                                                                                                                                                                                                   - - `NOTION_INTEL_CARDS_DB` - Intel Cards database ID
                                                                                                                                                                                                                     - - `NOTION_JOURNAL_DB` - Journal database ID
                                                                                                                                                                                                                      
                                                                                                                                                                                                                       - ---
                                                                                                                                                                                                                       
                                                                                                                                                                                                                       ## Database Relationships
                                                                                                                                                                                                                       
                                                                                                                                                                                                                       ```
                                                                                                                                                                                                                       ┌─────────────┐
                                                                                                                                                                                                                       │  Watchlist  │
                                                                                                                                                                                                                       │   (User)    │
                                                                                                                                                                                                                       └──────┬──────┘
                                                                                                                                                                                                                              │ 1:many
                                                                                                                                                                                                                              ▼
                                                                                                                                                                                                                       ┌─────────────┐      ┌─────────────┐
                                                                                                                                                                                                                       │ Intel Cards │◄────►│   Journal   │
                                                                                                                                                                                                                       │ (Generated) │      │  (Alerts)   │
                                                                                                                                                                                                                       └─────────────┘      └─────────────┘
                                                                                                                                                                                                                       ```
                                                                                                                                                                                                                       
                                                                                                                                                                                                                       - **Watchlist → Intel Cards**: One ticker can have many Intel Cards over time
                                                                                                                                                                                                                       - - **Watchlist → Journal**: One ticker can have many alerts logged
                                                                                                                                                                                                                         - - **Intel Cards ↔ Journal**: Alerts can reference the Intel Card that was generated
                                                                                                                                                                                                                          
                                                                                                                                                                                                                           - ---
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ## Sample Data
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ### Sample Watchlist Entry
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ```json
                                                                                                                                                                                                                           {
                                                                                                                                                                                                                             "Ticker": "AAPL",
                                                                                                                                                                                                                             "Company Name": "Apple Inc.",
                                                                                                                                                                                                                             "Added Date": "2024-01-15",
                                                                                                                                                                                                                             "Status": "Active",
                                                                                                                                                                                                                             "Thesis": "Strong services growth, AI potential",
                                                                                                                                                                                                                             "Target Price": 200,
                                                                                                                                                                                                                             "Stop Loss": 160,
                                                                                                                                                                                                                             "Sector": "Technology",
                                                                                                                                                                                                                             "Alert Priority": "High",
                                                                                                                                                                                                                             "Tags": ["FAANG", "Dividend"]
                                                                                                                                                                                                                           }
                                                                                                                                                                                                                           ```
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ### Sample Intel Card Entry
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ```json
                                                                                                                                                                                                                           {
                                                                                                                                                                                                                             "Title": "AAPL - 2024-01-20",
                                                                                                                                                                                                                             "Ticker": "AAPL",
                                                                                                                                                                                                                             "Generated At": "2024-01-20T14:30:00Z",
                                                                                                                                                                                                                             "Trigger": "On-Demand",
                                                                                                                                                                                                                             "Current Price": 185.50,
                                                                                                                                                                                                                             "Day Change %": 2.3,
                                                                                                                                                                                                                             "Market Cap": 2850000000000,
                                                                                                                                                                                                                             "PE Ratio": 28.5,
                                                                                                                                                                                                                             "Risk Score": 4,
                                                                                                                                                                                                                             "Risk Flags": ["Earnings Imminent"],
                                                                                                                                                                                                                             "AI Summary": "AAPL showing strength ahead of earnings..."
                                                                                                                                                                                                                           }
                                                                                                                                                                                                                           ```
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ### Sample Journal Entry
                                                                                                                                                                                                                           
                                                                                                                                                                                                                           ```json
                                                                                                                                                                                                                           {
                                                                                                                                                                                                                             "Title": "AAPL moved +3.5%",
                                                                                                                                                                                                                             "Timestamp": "2024-01-20T10:45:00Z",
                                                                                                                                                                                                                             "Ticker": "AAPL",
                                                                                                                                                                                                                             "Alert Type": "Price Move",
                                                                                                                                                                                                                             "Severity": "Medium",
                                                                                                                                                                                                                             "Trigger Value": "3.5%",
                                                                                                                                                                                                                             "Threshold": "3%",
                                                                                                                                                                                                                             "Channel": "Discord",
                                                                                                                                                                                                                             "Acknowledged": false
                                                                                                                                                                                                                           }
                                                                                                                                                                                                                           ```
