Phase 1: News Collection & Processing
Step 1.1: News Aggregator Module

Set up news sources APIs (Reuters, Bloomberg, Financial Times, CNBC, MarketWatch)
Implement 10-second polling mechanism
Filter news by relevance (keywords: target assets, earnings, CEO announcements, regulatory changes)
Store raw news with timestamp in database (SQLite or PostgreSQL)

Step 1.2: News Deduplication

Check for duplicate/similar stories from different sources
Keep only unique news items
Track which assets each news item mentions


Phase 2: AI Analysis Engine
Step 2.1: News Summarization

Send raw news article to Claude API
Get concise 2-3 sentence summary
Extract key facts (who, what, when, impact)

Step 2.2: Market Impact Analysis

Send summarized news + asset context to Claude API
Request structured JSON response with:

json  {
    "affected_assets": ["AAPL", "MSFT"],
    "direction": "BUY" | "SELL" | "NEUTRAL",
    "confidence_percent": 0-100,
    "expected_move_percent": float,
    "reasoning": "string",
    "time_horizon": "minutes|hours|days",
    "sentiment_score": -1 to +1
  }

Use prompt engineering with:

Historical context about the asset
Current market conditions
Similar past news outcomes (if available)



Step 2.3: Position Sizing Calculation

Send to Claude API:

Total portfolio value
Available cash
Current positions
Confidence level from Step 2.2
Risk tolerance (e.g., max 2% per trade)


Get recommended position size in USD


Phase 3: Signal Dashboard (Frontend)
Step 3.1: Real-time Signal Display

Web-based dashboard (React or similar)
Show live feed of:

Latest news items
AI analysis results
Recommended trades
Confidence levels


Color-coded signals (green=BUY, red=SELL, gray=NEUTRAL)

Step 3.2: Trade Approval Interface

For each signal, show:

News headline & summary
Asset name & current price
AI reasoning
Recommended action (BUY/SELL)
Position size
Stop-loss & take-profit levels


Buttons: "Execute", "Reject", "Modify"

Step 3.3: Portfolio Overview

Current holdings (simulated)
P&L (profit/loss)
Trade history
Win rate statistics


Phase 4: Paper Trading Engine
Step 4.1: Simulated Execution

Track virtual portfolio (start with e.g., $65,000)
When user approves trade:

Record entry price (use real-time market data)
Calculate shares purchased
Set stop-loss (-2%) and take-profit (+2%)
Deduct from available cash



Step 4.2: Position Monitoring

Poll current prices every 10-30 seconds
Check if stop-loss or take-profit hit
Auto-close positions when targets reached
Record exit price and P&L

Step 4.3: Trade Logging

Store all trades in database:

Entry/exit timestamps
Asset
Direction
Entry/exit prices
P&L amount & percentage
AI confidence level
News headline that triggered it




Phase 5: Performance Analytics
Step 5.1: Statistics Dashboard

Total trades executed
Win rate (% profitable)
Average gain per winning trade
Average loss per losing trade
Total P&L
Sharpe ratio (if applicable)
Compare vs buy-and-hold benchmark

Step 5.2: AI Confidence Calibration

Analyze: When AI says 70% confidence, is it actually 70% accurate?
Create calibration chart
Suggest confidence threshold adjustments


Phase 6: Risk Management & Safeguards
Step 6.1: Circuit Breakers

Daily loss limit (e.g., stop trading if -5% portfolio value)
Maximum concurrent positions (e.g., max 5)
Per-trade position limit (e.g., max 10% of portfolio)

Step 6.2: Alerts & Notifications

Email/SMS when trade executed
Alert when stop-loss or take-profit hit
Daily performance summary


Tech Stack Recommendations
Backend:

Python (FastAPI or Flask)
Database: PostgreSQL or SQLite
News APIs: NewsAPI, Alpha Vantage, or scraping
Market data: yfinance, Alpha Vantage, or IEX Cloud
AI: Anthropic Claude API

Frontend:

React + Tailwind CSS
Real-time updates: WebSockets or Server-Sent Events
Charts: Recharts or Chart.js

Infrastructure:

Run locally or on AWS/Heroku
Scheduler: APScheduler or Celery for polling


Monitored Assets (Starting List)
Tech Stocks:

AAPL, MSFT, NVDA, GOOGL, META, AMZN, TSLA

Finance:

JPM, BAC, V, MA

Crypto (if eToro supports):

BTC, ETH

ETFs:

SPY, QQQ


Important Notes for Cursor

Start with paper trading only - no real money
Log everything - every decision, every price, every AI response
Make prompts modular - easy to A/B test different prompt strategies
Error handling - APIs fail, handle gracefully
Rate limits - respect Claude API and news API limits
Security - API keys in environment variables, never commit


Success Metrics (After 30 Days)

Win rate > 45%
Average trade duration
AI confidence vs actual accuracy correlation
Compare paper portfolio vs buy-and-hold SPY


Next Steps After POC

Implement eToro API integration (replace paper trading)
Add more sophisticated AI models (fine-tuned on historical data)
Implement backtesting on historical news
Add sentiment analysis from Twitter/Reddit
Multi-timeframe analysis


Start with Phase 1-3 for MVP. Get the news → AI analysis → dashboard loop working first before adding execution logic.
