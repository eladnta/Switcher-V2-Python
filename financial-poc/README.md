# Financial Analysis & Paper Trading System (POC)

A cross-asset analysis and paper-trading system. Starts with a virtual $10,000,
allocates by risk and conviction, trades at realistic human pace (minutes-to-days,
not HFT), and measures its own prediction accuracy over time — then compares
itself against the market (SPY/QQQ).

> **Private use.** Built entirely on free/public data (Yahoo Finance, SEC EDGAR).
> No API keys required for the POC. Everything runs locally; nothing is sent out.
> Not financial advice.

## Architecture

```
                          ┌──────────────┐
                          │  engine.py   │  ← central orchestrator
                          │  analyze()   │     routes by asset class
                          └──────┬───────┘
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
      ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
      │  EQUITY      │  │  ETF/INDEX   │  │  SIGNAL OVERLAY   │
      │  fundamental │  │  COMMODITY   │  │  news + macro +   │
      │  risk/horizon│  │  CURRENCY    │  │  alternatives →   │
      │  /analyst    │  │  BOND/CRYPTO │  │  per-STOCK impact │
      │              │  │  technical   │  │                   │
      └──────────────┘  └──────────────┘  └──────────────────┘
              └──────────────────┬──────────────────┘
                                 ▼
                    Unified Recommendation (same shape)
                                 ▼
              ┌──────────────────────────────────┐
              │  TRADER  position-sizing → order │
              │  engine (delay+slippage) → P&L   │
              │  conviction tracker → benchmark  │
              └──────────────────────────────────┘
```

## Tradeable Universe

Beyond individual stocks, the system trades **indices, sectors, countries,
commodities, currencies, bonds, and crypto** — all via ETFs on US exchanges.
This also gives multi-country exposure (Japan `EWJ`, Germany `EWG`, India
`INDA`, Israel `EIS`) without foreign data feeds.

| Asset class | Examples | Analysis |
|---|---|---|
| Equity | AAPL, NVDA, JPM | Fundamental: risk + horizon + analyst consensus |
| Index ETF | SPY, QQQ, IWM | Technical: trend + momentum + relative strength |
| Country ETF | EWJ, EWG, INDA, EIS | Technical + country macro |
| Sector ETF | XLK, XLF, XLE | Technical + sector rotation |
| Commodity | GLD, USO, CPER | Technical + supply/macro |
| Currency | UUP, FXE, FXY | Technical + rate-differential |
| Bond | TLT, HYG, TIP | Technical + rate regime |
| Crypto | BTC-USD, ETH-USD | Technical + risk appetite |

## Modules

| Module | Role |
|---|---|
| `universe.py` | Registry of all tradeable instruments + asset-class routing |
| `engine.py` | Central orchestrator — analyze any symbol, apply signal overlay |
| `analysis/` | Equity scorers (risk, horizon, analyst) + technical analyzer |
| `signals/` | Macro monitor, news analyzer, alternatives, stock-impact, event DB |
| `trading/` | Portfolio, position sizer, order engine, conviction tracker, benchmark |
| `trader.py` | Paper-trading CLI |
| `monitor.py` | Continuous signal monitoring CLI |
| `demo.py` | Offline demo with mock data |

## Usage

```bash
pip install -r requirements.txt

# Analysis
python main.py analyze AAPL          # single equity, full report
python monitor.py signals            # market-wide macro + alternatives snapshot
python monitor.py stock XOM          # full signal overlay for one stock
python monitor.py news NVDA          # news + entity sentiment + event tags

# Paper trading ($10,000 virtual)
python trader.py demo                # dry-run with mock data
python trader.py run                 # full cycle (mixed equity + cross-asset)
python trader.py run --universe cross-asset   # ETFs/commodities/FX/crypto only
python trader.py status              # portfolio + P&L
python trader.py performance         # Sharpe, drawdown, alpha vs SPY/QQQ
python trader.py conviction          # prediction accuracy by horizon/sector/risk

# Continuous monitoring
python monitor.py run --interval 30  # refresh signals every 30 min
python monitor.py db                 # event database stats
```

## Design Principles

1. **Realistic pace** — orders execute after a 5–45 min human delay + slippage.
   Horizons span minutes to months, never HFT.
2. **Gets smarter over time** — the SQLite event DB caches signals (faster each
   run) and accumulates event→outcome history (accuracy patterns per
   event-type/sector).
3. **Reliability over hype** — the goal is calibrated, measurable prediction
   accuracy, benchmarked against the market.

## Roadmap

- [ ] Fundamental alpha signals (PEAD, analyst revisions, insider Form 4, accruals)
- [ ] Point-in-time fundamentals from EDGAR (avoid lookahead bias)
- [ ] Calibration layer (Brier score) + adaptive signal weighting
- [ ] LLM filing reader (10-K/earnings call) — first paid module, ROI-gated
- [ ] Web dashboard (portfolio vs market, signal heatmap, event timeline)
