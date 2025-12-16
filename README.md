# AI-Driven Trading Signal System

> A hybrid AI-powered trading system that analyzes real-time news, generates trade signals, and provides human-in-the-loop decision making.

## 🎯 Project Overview

This system monitors financial news from multiple sources, uses AI to analyze potential market impact, and generates trading signals for stocks and crypto. It starts in **paper trading mode** where all trades are simulated, allowing you to validate the system before risking real capital.

### Key Features

- ✅ Real-time news aggregation from major financial sources
- ✅ AI-powered market impact analysis using Claude API
- ✅ Automated position sizing recommendations
- ✅ Human approval required before trade execution
- ✅ Paper trading simulation engine
- ✅ Performance analytics and win rate tracking
- ✅ Risk management with circuit breakers
- ✅ Real-time dashboard for monitoring signals

## 🏗️ Architecture

```
News Sources → Aggregator → AI Analysis → Signal Dashboard → Paper Trading Engine
                                ↓
                          Trade Logger → Analytics
```

## 📋 Table of Contents

- [Phase 1: News Collection & Processing](#phase-1-news-collection--processing)
- [Phase 2: AI Analysis Engine](#phase-2-ai-analysis-engine)
- [Phase 3: Signal Dashboard](#phase-3-signal-dashboard)
- [Phase 4: Paper Trading Engine](#phase-4-paper-trading-engine)
- [Phase 5: Performance Analytics](#phase-5-performance-analytics)
- [Phase 6: Risk Management](#phase-6-risk-management)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Monitored Assets](#monitored-assets)
- [Success Metrics](#success-metrics)

---

## Phase 1: News Collection & Processing

### 1.1 News Aggregator Module

**Objective:** Collect real-time financial news from multiple sources.

**Requirements:**

- Set up API connections to:
  - Reuters
  - Bloomberg
  - Financial Times
  - CNBC
  - MarketWatch
  - Seeking Alpha
- Implement 10-second polling mechanism
- Filter news by keywords (target assets, earnings, CEO announcements, regulatory changes)
- Store raw news in database with:
  - Timestamp
  - Source
  - Headline
  - Content
  - Mentioned assets

**Database Schema:**

```sql
CREATE TABLE news_items (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP,
    source VARCHAR(50),
    headline TEXT,
    content TEXT,
    url TEXT,
    mentioned_assets TEXT[],
    processed BOOLEAN DEFAULT FALSE
);
```

### 1.2 News Deduplication

**Objective:** Avoid processing the same story multiple times.

**Requirements:**

- Compare incoming news against recent items (last 1 hour)
- Use fuzzy matching on headlines (e.g., 80% similarity threshold)
- Mark duplicates in database
- Keep only the first occurrence from the highest-priority source

---

## Phase 2: AI Analysis Engine

### 2.1 News Summarization

**Objective:** Convert raw news articles into concise summaries.

**Implementation:**

- Send raw news to Claude API
- Prompt: Extract key facts (who, what, when, impact)
- Store summary in database (max 2-3 sentences)

**Example Prompt:**

```
Summarize this financial news in 2-3 sentences. Focus on:
- Which company/asset is affected
- What happened
- Potential market impact

News: {article_content}
```

### 2.2 Market Impact Analysis

**Objective:** Determine if news will move markets and in which direction.

**Implementation:**

- Send summarized news + asset context to Claude API
- Request structured JSON response

**Required Output Format:**

```json
{
  "affected_assets": ["AAPL", "MSFT"],
  "direction": "BUY",
  "confidence_percent": 70,
  "expected_move_percent": 2.0,
  "reasoning": "Apple CEO announced breakthrough in AI chips, historically such announcements lead to 1-3% gains",
  "time_horizon": "hours",
  "sentiment_score": 0.8
}
```

**Prompt Template:**

```
You are a financial analyst. Analyze this news and predict market impact.

News Summary: {summary}
Asset: {asset_symbol}
Current Price: {current_price}
Recent Performance: {recent_performance}

Respond ONLY with valid JSON containing:
- affected_assets: list of ticker symbols
- direction: "BUY", "SELL", or "NEUTRAL"
- confidence_percent: 0-100
- expected_move_percent: estimated price change
- reasoning: brief explanation (max 50 words)
- time_horizon: "minutes", "hours", or "days"
- sentiment_score: -1 (very negative) to +1 (very positive)
```

### 2.3 Position Sizing Calculation

**Objective:** Determine how much capital to allocate to each trade.

**Implementation:**

- Input:
  - Total portfolio value
  - Available cash
  - Current open positions
  - Confidence level from 2.2
  - Risk tolerance (default: max 2% risk per trade)
- Output: Recommended position size in USD

**Prompt Template:**

```
Calculate position size for this trade:

Portfolio Value: ${portfolio_value}
Available Cash: ${available_cash}
Current Positions: {current_positions}
Trade Confidence: {confidence_percent}%
Max Risk Per Trade: 2%
Expected Move: {expected_move_percent}%

Provide position size in USD. Consider:
- Higher confidence = larger position (but never exceed 10% of portfolio)
- Account for existing exposure to similar assets
- Leave cash buffer for other opportunities
```

---

## Phase 3: Signal Dashboard

### 3.1 Real-time Signal Display

**Objective:** Web interface showing live AI analysis results.

**Features:**

- Live news feed with timestamps
- AI analysis cards showing:
  - Asset symbol
  - Direction (BUY/SELL/NEUTRAL)
  - Confidence level (progress bar)
  - Expected move %
  - AI reasoning
- Color coding:
  - 🟢 Green: BUY signals
  - 🔴 Red: SELL signals
  - ⚪ Gray: NEUTRAL
- Auto-refresh every 10 seconds

**UI Components:**

```
┌─────────────────────────────────────────┐
│  AI Trading Signal Dashboard            │
├─────────────────────────────────────────┤
│  Latest Signals                         │
│  ┌────────────────────────────────────┐ │
│  │ AAPL - BUY                         │ │
│  │ Confidence: ████████░░ 80%         │ │
│  │ Expected: +2.1%                    │ │
│  │ Reason: CEO announced AI chip      │ │
│  │ [Execute] [Reject] [Details]       │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### 3.2 Trade Approval Interface

**Objective:** Human-in-the-loop decision making.

**Features:**

- For each signal, display:
  - Full news headline
  - Asset current price
  - Recommended action
  - Position size ($)
  - Number of shares
  - Entry price
  - Stop-loss price (-2%)
  - Take-profit price (+2%)
- Action buttons:
  - ✅ Execute: Approve and execute trade
  - ❌ Reject: Dismiss signal
  - ✏️ Modify: Adjust position size or levels

### 3.3 Portfolio Overview

**Objective:** Monitor current holdings and performance.

**Display:**

- Total portfolio value
- Available cash
- Open positions table:
  - Asset
  - Quantity
  - Entry price
  - Current price
  - P&L ($)
  - P&L (%)
- Closed trades history
- Overall win rate
- Total P&L

---

## Phase 4: Paper Trading Engine

### 4.1 Simulated Execution

**Objective:** Execute trades in simulation mode.

**Starting Capital:** $65,000 (configurable)

**On Trade Approval:**

1. Fetch current real-time price for asset
2. Calculate shares = position_size_usd / current_price
3. Record trade in database:
   - Entry timestamp
   - Asset symbol
   - Direction (BUY/SELL)
   - Quantity
   - Entry price
   - Position size
   - Stop-loss level (entry_price \* 0.98)
   - Take-profit level (entry_price \* 1.02)
4. Deduct position size from available cash
5. Add to open positions

**Database Schema:**

```sql
CREATE TABLE trades (
    id SERIAL PRIMARY KEY,
    entry_time TIMESTAMP,
    exit_time TIMESTAMP,
    asset VARCHAR(10),
    direction VARCHAR(4),
    quantity DECIMAL,
    entry_price DECIMAL,
    exit_price DECIMAL,
    stop_loss DECIMAL,
    take_profit DECIMAL,
    position_size DECIMAL,
    pnl DECIMAL,
    pnl_percent DECIMAL,
    confidence_level INT,
    news_headline TEXT,
    status VARCHAR(20) -- 'OPEN', 'CLOSED_TP', 'CLOSED_SL', 'CLOSED_MANUAL'
);
```

### 4.2 Position Monitoring

**Objective:** Auto-close positions when targets hit.

**Implementation:**

- Poll current prices every 30 seconds
- For each open position, check:
  - If current_price <= stop_loss → Close position (loss)
  - If current_price >= take_profit → Close position (gain)
- On position close:
  - Calculate P&L = (exit_price - entry_price) \* quantity
  - Update trade record with exit details
  - Add position_size + pnl back to available cash
  - Move to closed positions

**Monitoring Logic:**

```python
def monitor_positions():
    for position in get_open_positions():
        current_price = get_current_price(position.asset)

        if current_price <= position.stop_loss:
            close_position(position, current_price, reason='STOP_LOSS')
        elif current_price >= position.take_profit:
            close_position(position, current_price, reason='TAKE_PROFIT')
```

### 4.3 Trade Logging

**Objective:** Comprehensive audit trail of all activities.

**Logged Data:**

- Every news item processed
- Every AI analysis result
- Every signal generated
- Every trade decision (approved/rejected)
- Every position opened/closed
- Price at every decision point

---

## Phase 5: Performance Analytics

### 5.1 Statistics Dashboard

**Key Metrics:**

- **Total Trades:** Count of executed trades
- **Win Rate:** % of profitable trades
- **Average Win:** Mean profit on winning trades
- **Average Loss:** Mean loss on losing trades
- **Profit Factor:** Total wins / Total losses
- **Total P&L:** Net profit/loss ($)
- **Total P&L %:** Return on initial capital
- **Max Drawdown:** Largest peak-to-trough decline
- **Sharpe Ratio:** Risk-adjusted return (if applicable)

**Benchmark Comparison:**

- Compare against buy-and-hold SPY
- Show side-by-side performance chart

**Visualization:**

- Equity curve (portfolio value over time)
- Win rate by asset
- P&L distribution histogram
- Confidence level vs actual outcome scatter plot

### 5.2 AI Confidence Calibration

**Objective:** Validate AI accuracy predictions.

**Analysis:**

- Group trades by confidence level (0-20%, 20-40%, etc.)
- Calculate actual win rate for each group
- Create calibration curve:
  - X-axis: AI confidence
  - Y-axis: Actual win rate
- Ideal: AI says 70% → actually wins 70% of time

**Output:**

- Calibration chart
- Recommendation: "AI is overconfident, lower threshold from 70% to 60%"

---

## Phase 6: Risk Management & Safeguards

### 6.1 Circuit Breakers

**Safety Limits:**

1. **Daily Loss Limit**

   - Stop all trading if daily P&L < -5% of portfolio
   - Require manual override to resume

2. **Maximum Concurrent Positions**

   - Max 5 open positions at once
   - Prevents over-diversification

3. **Per-Trade Position Limit**

   - No single trade > 10% of portfolio
   - Even with 99% AI confidence

4. **Consecutive Loss Limit**

   - After 3 consecutive losses, reduce position sizes by 50%
   - After 5 consecutive losses, pause trading for human review

5. **API Rate Limits**
   - Respect Claude API rate limits
   - Queue requests if approaching limit

### 6.2 Alerts & Notifications

**Notification Triggers:**

- Trade executed (email/SMS)
- Stop-loss hit
- Take-profit hit
- Circuit breaker activated
- API error or system failure
- Daily performance summary (EOD)

**Channels:**

- Email
- SMS (via Twilio)
- In-dashboard alerts

---

## Tech Stack

### Backend

- **Language:** Python 3.10+
- **Framework:** FastAPI
- **Database:** PostgreSQL (production) or SQLite (development)
- **Task Scheduler:** APScheduler or Celery
- **AI API:** Anthropic Claude API
- **Market Data:**
  - yfinance (free, good for testing)
  - Alpha Vantage (free tier available)
  - IEX Cloud (paid, more reliable)
- **News APIs:**
  - NewsAPI
  - Alpha Vantage News
  - Custom web scraping (BeautifulSoup)

### Frontend

- **Framework:** React 18+ with Vite
- **Styling:** Tailwind CSS
- **Charts:** Recharts or Chart.js
- **State Management:** React Context or Zustand
- **Real-time Updates:** WebSockets or Server-Sent Events

### Infrastructure

- **Development:** Run locally
- **Production:** AWS EC2, Heroku, or DigitalOcean
- **Environment:** Docker + Docker Compose
- **Secrets Management:** .env files + environment variables

### Required APIs & Keys

- Anthropic Claude API key
- News API key(s)
- Market data API key(s)
- eToro API credentials (Phase 7+)

---

## Getting Started

### Prerequisites

```bash
- Python 3.10+
- Node.js 18+
- PostgreSQL (or SQLite for development)
- Git
```

### Installation

1. **Clone Repository**

```bash
git clone <repo-url>
cd ai-trading-signals
```

2. **Backend Setup**

```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. **Environment Variables**

```bash
cp .env.example .env
# Edit .env with your API keys:
# ANTHROPIC_API_KEY=your_key_here
# NEWS_API_KEY=your_key_here
# MARKET_DATA_API_KEY=your_key_here
```

4. **Database Setup**

```bash
python init_db.py  # Creates tables
```

5. **Frontend Setup**

```bash
cd ../frontend
npm install
```

6. **Run Development Servers**

```bash
# Terminal 1 - Backend
cd backend
python main.py

# Terminal 2 - Frontend
cd frontend
npm run dev
```

7. **Access Dashboard**

```
Open browser to: http://localhost:5173
```

### Configuration

Edit `config.py` to customize:

- Polling interval (default: 10 seconds)
- Monitored assets
- Risk parameters (stop-loss, take-profit)
- Position sizing rules
- Circuit breaker thresholds

---

## Monitored Assets

### Initial Asset List

**Tech Stocks (High Volatility):**

- AAPL - Apple
- MSFT - Microsoft
- NVDA - NVIDIA
- GOOGL - Google
- META - Meta
- AMZN - Amazon
- TSLA - Tesla

**Financial Stocks (Moderate Volatility):**

- JPM - JPMorgan Chase
- BAC - Bank of America
- V - Visa
- MA - Mastercard

**Crypto (Very High Volatility):**

- BTC - Bitcoin
- ETH - Ethereum

**ETFs (Lower Volatility):**

- SPY - S&P 500 ETF
- QQQ - Nasdaq-100 ETF

**Note:** Start with 5-10 assets for MVP, expand after validation.

---

## Success Metrics

### After 30 Days of Paper Trading

**Minimum Acceptable Performance:**

- Win rate: **> 45%**
- Total trades executed: **> 50**
- AI confidence calibration: **< 15% error**
- System uptime: **> 95%**

**Target Performance:**

- Win rate: **> 52%**
- Profit factor: **> 1.5**
- Beat buy-and-hold SPY: **Yes**
- Max drawdown: **< 15%**

**Red Flags (Do NOT go live if):**

- Win rate < 40%
- AI confidence wildly miscalibrated (70% confidence = 30% actual)
- Large consecutive losses without recovery
- System crashes or data gaps

---

## Development Roadmap

### Phase 1-3 (Week 1-2): MVP

- ✅ News aggregation working
- ✅ AI analysis generating signals
- ✅ Dashboard displaying signals
- ✅ Manual trade approval

### Phase 4 (Week 3): Paper Trading

- ✅ Simulated execution
- ✅ Position monitoring
- ✅ Auto-close on targets

### Phase 5 (Week 4): Analytics

- ✅ Performance dashboard
- ✅ Win rate tracking
- ✅ Calibration analysis

### Phase 6 (Week 5): Risk Management

- ✅ Circuit breakers
- ✅ Alerts & notifications
- ✅ Error handling

### Phase 7 (Future): Live Trading

- ⏳ eToro API integration
- ⏳ Real money execution
- ⏳ Advanced strategies

### Phase 8 (Future): Enhancements

- ⏳ Backtesting engine
- ⏳ Fine-tuned AI model
- ⏳ Sentiment analysis (Twitter/Reddit)
- ⏳ Multi-timeframe analysis

---

## Project Structure

```
ai-trading-signals/
├── backend/
│   ├── main.py                 # FastAPI app entry point
│   ├── config.py               # Configuration settings
│   ├── database.py             # Database connection
│   ├── models.py               # SQLAlchemy models
│   ├── news/
│   │   ├── aggregator.py       # News collection
│   │   └── deduplicator.py     # Duplicate detection
│   ├── ai/
│   │   ├── summarizer.py       # News summarization
│   │   ├── analyzer.py         # Market impact analysis
│   │   └── position_sizer.py   # Position sizing
│   ├── trading/
│   │   ├── paper_engine.py     # Simulated execution
│   │   └── monitor.py          # Position monitoring
│   ├── analytics/
│   │   └── performance.py      # Statistics calculation
│   └── utils/
│       ├── market_data.py      # Price fetching
│       └── notifications.py    # Alerts
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Dashboard.jsx
│   │   │   ├── SignalCard.jsx
│   │   │   ├── Portfolio.jsx
│   │   │   └── Analytics.jsx
│   │   ├── App.jsx
│   │   └── main.jsx
│   └── package.json
├── .env.example
├── requirements.txt
├── README.md
└── docker-compose.yml
```

---

## Important Notes

### ⚠️ Risk Warnings

1. **This is for educational/research purposes**
2. **Past performance ≠ future results**
3. **AI can be wrong - often spectacularly wrong**
4. **Never invest money you can't afford to lose**
5. **Start with paper trading for at least 1-3 months**
6. **Consult a financial advisor before live trading**

### 🔒 Security Best Practices

- Never commit API keys to Git
- Use environment variables for secrets
- Implement rate limiting on endpoints
- Validate all user inputs
- Log all actions for audit trail
- Encrypt sensitive data in database

### 📝 Logging Strategy

- Log level: INFO for normal operation, DEBUG for development
- Log all AI prompts and responses
- Log every price fetch and trade decision
- Rotate logs daily
- Keep logs for at least 90 days

---

## Troubleshooting

### Common Issues

**Issue:** News API rate limit exceeded

- **Solution:** Increase polling interval or upgrade API tier

**Issue:** AI returns invalid JSON

- **Solution:** Add JSON validation and retry logic with stricter prompts

**Issue:** Prices not updating

- **Solution:** Check market hours and API status

**Issue:** High latency (> 30 seconds)

- **Solution:** Optimize database queries, add caching

---

## Contributing

This is a personal project, but best practices:

1. Create feature branches
2. Write tests for critical logic
3. Document all functions
4. Update README with new features

---

## License

This project is for personal use. Not financial advice.

---

## Next Steps

1. **Set up development environment** (Prerequisites)
2. **Implement Phase 1** (News aggregation)
3. **Test with sample news** (Verify AI analysis)
4. **Build dashboard** (Visualize signals)
5. **Run paper trading for 30 days** (Validate strategy)
6. **Review metrics** (Decide on live trading)

---

## Contact & Support

For questions or issues:

- Check logs in `./logs/`
- Review database for data integrity
- Test AI prompts in Claude.ai first

**Remember:** This is experimental. Trade responsibly. 🚀
