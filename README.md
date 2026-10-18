# Lyman-Alpha Trading Terminal

### A market-intelligence terminal built for the Skyloong GK104Pro keyboard screen.

Lyman-Alpha Trading Terminal is a local Python-based market-monitoring and screening system designed specifically to make use of the **Skyloong GK104Pro keyboard's integrated 320×240 screen**.

The project turns the keyboard screen into a dedicated market-information display while providing a full analytical terminal through a local web dashboard.

![Dashboard — Market Monitor](screenshots/01-dashboard.png)

The concept is to separate the experience into two layers:

**PC / Web Terminal**

Full analysis, screening, charts, watchlists, news, earnings and alert management.

**Skyloong GK104Pro Screen**

A continuously updating, glanceable market surface showing watchlist data, top movers, mini price charts and triggered alerts.

This creates a small, dedicated trading-information terminal that sits directly on the user's desk rather than requiring a browser window to remain open simply to monitor markets.

---

## Why Lyman-Alpha?

Traditional market dashboards are designed around large displays and require the user to actively look at a browser or trading application.

Lyman-Alpha takes a different approach.

The **Skyloong GK104Pro keyboard screen becomes a persistent secondary market display**.

The terminal continuously determines what information deserves attention and renders the appropriate screen automatically.

Instead of attempting to place an entire trading dashboard onto a small 320×240 panel, Lyman-Alpha uses the limited screen real estate deliberately:

```text
MARKET DATA
     ↓
WATCHLIST
     ↓
MOVER DETECTION
     ↓
PAGE SCHEDULER
     ↓
SCREEN RENDERER
     ↓
SKYLOONG GK104Pro
```

The keyboard screen therefore acts as an always-visible market information layer, while the browser remains the detailed analytical workspace.

---

# Key Capabilities

## 1. Skyloong GK104Pro Integration

Lyman-Alpha is built around the **Skyloong GK104Pro keyboard screen**.

The application generates the content displayed on the keyboard's screen and communicates with the configured device over the local network.

The display engine manages:

- Device IP
- Device connection state
- Display page rotation
- Alert pages
- Watchlist pages
- Top-mover pages
- JPEG rendering
- Device push
- Device error states
- Last successful push
- Confirmed device target

The application also maintains a distinction between the locally generated display preview and a successfully confirmed device push.

**Generated preview** — the movers-first rotation rendered for the 320×240 screen, with a configurable rotation plan:

![SkyLoong display preview](screenshots/05-device-display.png)

**On the hardware** — the live top-movers page pushed to the physical SkyLoong GK104Pro keyboard screen:

![SkyLoong 4.0 hardware display](screenshots/06-skyloong-hardware.jpg)

---

## 2. Intelligent Screen Scheduling

A 320×240 screen cannot display an entire market terminal.

Lyman-Alpha therefore uses a scheduler to determine what should be displayed.

The normal display consists of rotating watchlist pages.

When significant movers are detected, the scheduler can prioritise a **TOP MOVERS** page before continuing through the watchlist.

Conceptually:

```text
┌─────────────────────────────┐
│       TOP MOVERS            │
│                             │
│ NVDA       +4.21%           │
│ TSLA       +3.18%           │
│ AMD        +2.87%           │
│ BTC-USD    +2.41%           │
└─────────────────────────────┘

              ↓

┌─────────────────────────────┐
│       WATCHLIST             │
│                             │
│ AAPL       258.41  +1.2%   │
│ MSFT       531.80  +0.8%   │
│ META       734.20  -0.4%   │
│ AMZN       231.12  +0.6%   │
└─────────────────────────────┘
```

The scheduler automatically determines the next page and retains the relevant symbols for rendering.

---

# 3. Market Data Engine

Lyman-Alpha retrieves market information through `yfinance`.

The quote engine can work with:

- Intraday data
- Historical daily data
- Previous close
- Current price
- Open
- High
- Low
- Volume
- Percentage change
- Absolute change
- Short-term momentum
- Market timestamp
- Timezone information

The application uses the latest trading session when processing intraday history and calculates short-term momentum from recent observations.

---

# 4. Streaming + Polling Architecture

Where supported by the installed `yfinance` implementation, Lyman-Alpha can use asynchronous WebSocket market-data streaming.

A polling mechanism provides a fallback when streaming is unavailable.

This gives the terminal two operating modes:

```text
                MARKET DATA
                     │
             ┌───────┴───────┐
             │               │
         WebSocket        Polling
             │               │
             └───────┬───────┘
                     ↓
               Quote Engine
                     ↓
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Web UI     Screener    GK104Pro
```

The streaming layer also uses timestamp ordering so older updates do not unnecessarily overwrite newer market information.

---

# 5. Watchlist

The watchlist is the central market universe used by the terminal.

Users can:

- Add symbols
- Remove symbols
- Rename instruments
- Reorder instruments
- Select instruments for detailed analysis

The watchlist also drives the physical GK104Pro display.

This means the same instruments being monitored in the browser can automatically become the instruments shown on the keyboard screen.

---

# 6. Mover Detection

Lyman-Alpha does not simply rotate symbols in a fixed order.

It calculates a mover score using daily price movement and short-term momentum:

```text
Mover Score =
|Daily Change %|
+
2 × |Short-Term Momentum %|
```

An instrument becomes a mover when it exceeds the configured daily-change or momentum threshold.

This allows the GK104Pro screen to behave more like an **attention engine** than a passive ticker.

The screen can surface the instruments currently experiencing meaningful movement rather than forcing the user to scan the entire watchlist manually.

---

# 7. Interactive Charts

The web terminal provides deeper historical analysis for the selected instrument.

Chart data can cover:

- 1D
- 5D
- 1M
- 6M
- 1Y
- 5Y

Shorter timeframes use higher-frequency market data while longer timeframes use increasingly aggregated historical data.

The application can also incorporate current quote information into the latest historical session.

---

# 8. Multi-Factor Stock Screener

The integrated screener is one of the main analytical components of Lyman-Alpha.

![Stock screener](screenshots/03-screener.png)

It is capable of combining:

### Price

- Direction
- Minimum change
- Maximum/minimum price
- Gap

### Volume

- Minimum volume
- Relative volume

### Technicals

- RSI
- SMA50
- SMA200
- Distance from 52-week high

### Fundamentals

- Market capitalisation
- P/E
- EPS
- Earnings growth
- Dividend yield
- Sector
- Provider rating
- Target-price-derived upside

### Events

- Recent news
- News keywords
- Upcoming earnings

The screening engine calculates these metrics for each candidate and only returns instruments satisfying the configured criteria.

---

# 9. Relative Volume

Relative volume is calculated against the preceding trading sessions.

Conceptually:

```text
Relative Volume =
Current Volume
÷
Average Previous 20 Sessions Volume
```

This allows the screener to distinguish between a price move occurring on ordinary activity and a move accompanied by unusually high participation.

---

# 10. Technical Analysis

The terminal currently calculates:

- RSI
- SMA50
- SMA200
- Daily change
- Gap %
- Relative volume
- Distance from 52-week high
- Short-term momentum

RSI is calculated using a Wilder-style smoothing approach rather than simply using a basic rolling average.

---

# 11. Fundamental Analysis

The screener can incorporate provider fundamental data including:

```text
Market Capitalisation
P/E
EPS
Quarterly Earnings Growth
Dividend Yield
Sector
Analyst Recommendation
Target Mean Price
```

It can also derive implied upside from the provider's target mean price:

```text
Upside % =
(Target Mean Price − Current Price)
÷ Current Price × 100
```

These metrics can be used individually or in combination with technical and market filters.

---

# 12. News Intelligence

Lyman-Alpha maintains an aggregated news feed for the watchlist.

News items are:

- Retrieved from the market-data provider
- Cached
- Timestamped
- Sorted newest-first
- Deduplicated
- Associated with relevant watchlist symbols

The system maintains a compact feed of the latest relevant headlines rather than treating every news request as an independent browser operation.

![Watchlist news and price alerts](screenshots/02-watchlist-alerts.png)

---

# 13. Earnings Monitoring

The terminal can identify upcoming earnings dates.

The screener can be configured to return instruments whose next earnings event falls within a specified number of days.

This makes it possible to combine market behaviour with event proximity.

For example:

```text
High Relative Volume
+
Positive Price Momentum
+
Upcoming Earnings
```

can form a screening condition for further research.

---

# 14. Price Alerts

Users can create price-based alerts.

![Price alerts](screenshots/04-price-alerts.png)

Supported conditions include:

```text
Price >= Target
Price <= Target
```

When an alert is triggered, the event is placed into the display queue.

The GK104Pro screen can temporarily interrupt its normal watchlist rotation and display the alert.

After the configured alert duration, the normal display schedule resumes.

This makes the keyboard screen useful even when the user is not actively looking at the web terminal.

---

# 15. The GK104Pro as an Alert Surface

The hardware integration is particularly useful for alerts.

Instead of:

```text
Browser
    ↓
Notification
    ↓
User notices
```

Lyman-Alpha can operate as:

```text
Market Event
     ↓
Alert Engine
     ↓
Display Queue
     ↓
GK104Pro Screen
     ↓
Persistent physical visibility
```

The display therefore becomes an extension of the terminal rather than simply a decorative screen.

---

# 16. Resilient Data Architecture

Market-data applications have to deal with incomplete responses, network failures and provider interruptions.

Lyman-Alpha includes a single-flight caching mechanism that prevents multiple simultaneous requests for the same resource and retains the last successful value when a transient fetch fails.

The application also tracks data errors separately from cached values so that a stale value is not silently treated as fresh data.

---

# 17. Concurrent Screening

The screener does not process a large universe strictly one symbol at a time.

It uses bounded worker pools to scan multiple instruments concurrently.

The scan engine tracks:

- Total symbols
- Completed symbols
- Failed symbols
- Current phase
- Scan duration
- Cancellation state

It also has a timeout mechanism and can retain previous results when a new scan fails.

This is particularly important when screening large universes where individual provider requests can fail independently.

---

# 18. Supported Screening Universes

The terminal can work with:

- Personal watchlist
- Custom symbols
- US momentum stocks
- US mega caps
- US high-beta stocks
- US index and sector ETFs
- Day gainers
- Day losers
- Most active
- Small-cap gainers
- Growth technology
- Undervalued growth
- Undervalued large caps
- Most shorted
- NIFTY 50 leaders
- FTSE 100 leaders
- Saudi Tadawul leaders
- Top crypto

Custom universes can also be supplied directly.

The resolved universe is capped at 250 symbols per scan.

---

# 19. Local-First Architecture

Lyman-Alpha runs as a local Flask application.

The dashboard is served through:

```text
http://127.0.0.1:5000
```

This keeps the primary terminal on the user's machine rather than requiring a cloud-hosted dashboard.

The Skyloong GK104Pro can then act as a network-connected display endpoint on the local network.

---

# 20. Architecture

```text
                         ┌───────────────────────┐
                         │  MARKET DATA PROVIDER  │
                         └───────────┬───────────┘
                                     │
                      ┌──────────────┴──────────────┐
                      │                             │
                WebSocket                       Polling
                      │                             │
                      └──────────────┬──────────────┘
                                     ↓
                              ┌───────────────┐
                              │  QUOTE ENGINE │
                              └───────┬───────┘
                                      │
              ┌───────────────────────┼──────────────────────┐
              │                       │                      │
              ↓                       ↓                      ↓
       ┌─────────────┐         ┌─────────────┐       ┌─────────────┐
       │   WEB UI    │         │   SCREENER  │       │ ALERT ENGINE│
       └──────┬──────┘         └─────────────┘       └──────┬──────┘
              │                                             │
              │                                             ↓
              │                                      ┌─────────────┐
              │                                      │ DISPLAY QUEUE│
              │                                      └──────┬──────┘
              │                                             │
              │                                             ↓
              │                                    ┌─────────────────┐
              │                                    │ DISPLAY ENGINE  │
              │                                    └────────┬────────┘
              │                                             │
              │                                      JPEG / HTTP
              │                                             │
              │                                             ↓
              │                                  ┌────────────────────┐
              │                                  │ SKYLOONG GK104PRO   │
              │                                  │ 320 × 240 SCREEN    │
              │                                  └────────────────────┘
              │
              ↓
       Full Analytical Terminal
```

---

# 21. Design Philosophy

The project is built around three principles.

### Information density

The GK104Pro screen is small, so the system prioritises information rather than attempting to reproduce a desktop terminal.

### Attention

The display should tell the user **what deserves attention**, particularly through movers and alerts.

### Separation of depth and immediacy

The browser provides analytical depth.

The keyboard screen provides immediate awareness.

Together they form a compact two-layer trading-information terminal.

---

# 22. Technology Stack

| Component | Technology |
|---|---|
| Language | Python |
| Web framework | Flask |
| Market data | yfinance |
| HTTP | Requests |
| Image rendering | Pillow |
| Charts | Lightweight Charts |
| Frontend | HTML / CSS / JavaScript |
| Hardware | Skyloong GK104Pro keyboard screen |
| Persistence | JSON |
| Concurrency | Python threading / concurrent futures |

---

# 23. Installation

```bash
pip install flask requests pillow yfinance
```

Run:

```bash
python skyloong_terminal.py
```

Open:

```text
http://127.0.0.1:5000
```

The application creates or loads its persistent configuration from:

```text
skyloong_config.json
```

---

# 24. Hardware Setup

1. Connect the Skyloong GK104Pro keyboard to the local network as required by the device.
2. Determine the keyboard screen/device IP.
3. Enter the IP address in the Lyman-Alpha configuration.
4. Enable the device from the terminal.
5. The display engine will begin generating and pushing display frames.
6. Verify the device status from the dashboard.

The application tracks the configured device target and confirms successful pushes before updating the device status.

---

# 25. What Makes This Different?

Lyman-Alpha is not simply:

> "A stock dashboard."

It is a **market-information system designed around a physical computing interface**.

The architecture connects:

```text
Financial Data
      +
Quantitative Filters
      +
Event Information
      +
Alert Logic
      +
Automated Rendering
      +
Dedicated Hardware
```

The result is a market terminal that can operate continuously in the background while the user works on the computer.

---

# 26. Limitations

Lyman-Alpha is a market-monitoring and research tool.

It is not:

- A brokerage terminal
- An order execution platform
- An institutional market-data terminal
- A guaranteed real-time market-data feed
- Investment advice

Market-data availability, latency and accuracy depend on the underlying provider and network conditions.

News is provider-based and polled rather than a guaranteed real-time news wire.

The physical display is controlled through the configured device interface; the application can confirm a successful push request but cannot independently verify what a person can physically see on the LCD.

---

# 27. Roadmap

Potential future development includes:

- Portfolio tracking
- Position-level P&L
- Broker integrations
- Additional market-data providers
- More technical indicators
- Valuation analytics
- Fundamental trend analysis
- Sector heatmaps
- Correlation analysis
- Risk analytics
- Backtesting
- Strategy modules
- Multiple watchlists
- Custom screener presets
- Excel/CSV export
- Multiple GK104Pro devices
- Additional hardware layouts
- Database-backed persistence
- Authentication
- Docker deployment

---

# 28. Disclaimer

Lyman-Alpha Trading Terminal is a software project for market monitoring, analysis and research.

Market information may be delayed, incomplete or unavailable.

Nothing displayed by the terminal constitutes investment, financial, tax or trading advice.

Users should independently verify market information before making financial decisions.

---

# 29. Project Status

**Current release: v2.1**

Lyman-Alpha currently combines:

```text
Market Data
      +
Watchlists
      +
Interactive Charts
      +
Technical Screening
      +
Fundamental Screening
      +
News
      +
Earnings
      +
Price Alerts
      +
Caching
      +
Persistent Configuration
      +
Skyloong GK104Pro Integration
```

into one local market-monitoring terminal.

---

## Built for the Skyloong GK104Pro

The project's hardware layer is specifically designed around the **Skyloong GK104Pro keyboard screen**.

The objective is not simply to put a clock, GIF or static information panel on the keyboard.

It is to make the keyboard itself part of a functioning market terminal.

The GK104Pro screen becomes the **always-visible market surface**.

The PC becomes the **analytical workstation**.

**Lyman-Alpha connects the two.**
