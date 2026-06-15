# Claude Project Context: nautilus_trader

This file provides codebase context for the **nautilus_trader** repository, used as part of an AI trading-knowledge project.

## Project Overview
* **Name**: NautilusTrader
* **Language**: Core engine in Rust + Cython, public API in Python
* **Type**: High-performance, event-driven algorithmic trading platform
* **Purpose**: Provide a single codebase for both backtesting and live (production) trading across multiple asset classes — equities, FX, futures, options, and crypto — with identical strategy code.
* **License**: LGPL-3.0 (check current license file — has changed between versions)

## Core System Boundaries
* **Event-driven architecture**: everything (market data, orders, fills, timers) flows through a message bus as events; strategies react to events rather than polling.
* **Backtest = Live parity**: the same `Strategy` class runs unchanged in `BacktestEngine` and in a live `TradingNode` — only the data/execution adapters differ.
* **Adapters**: venue-specific integrations live under `nautilus_trader/adapters/` (e.g. Binance, Interactive Brokers, Bybit, Databento) providing market data + order execution clients.
* **Nanosecond precision**: internal clock and timestamps use nanosecond resolution for accurate simulation of high-frequency strategies.

## Repository Structure & Key Directories
* `/nautilus_trader/` - Main Python/Cython package
  * `/nautilus_trader/core/` - Rust-backed core primitives (data types, time, UUID)
  * `/nautilus_trader/model/` - Domain model: instruments, orders, positions, currencies
  * `/nautilus_trader/trading/` - `Strategy` base class and trading logic
  * `/nautilus_trader/backtest/` - `BacktestEngine`, `BacktestNode` for historical simulation
  * `/nautilus_trader/live/` - `TradingNode` for live execution
  * `/nautilus_trader/adapters/` - Per-venue data/execution adapters (Binance, IB, Bybit, etc.)
  * `/nautilus_trader/indicators/` - Built-in technical indicators
* `/nautilus_core/` - Rust crates underlying the core engine
* `/examples/` - Backtest and live trading example scripts/strategies
* `/docs/` - Concepts (architecture, data, execution), tutorials, API reference

## Standard Operational Commands
* **Install**: `pip install -U nautilus_trader` (pre-built wheels; building from source requires Rust toolchain)
* **Run a backtest**: typically via a Python script instantiating `BacktestEngine`, adding data + venue, then `engine.run()`

## Standard Python API Usage (Strategy Blueprint)
```python
from nautilus_trader.trading.strategy import Strategy
from nautilus_trader.model.data import Bar

class MyStrategy(Strategy):
    def on_start(self):
        self.subscribe_bars(self.bar_type)

    def on_bar(self, bar: Bar):
        # strategy logic on each new bar
        if bar.close > bar.open:
            self.buy(...)

    def on_stop(self):
        self.close_all_positions(self.instrument_id)
```

## AI Assistant Guidelines for this Project
1. **Identical code for backtest/live**: when writing strategies, never hardcode backtest-only or live-only logic inside `Strategy` — configuration differences belong in the engine/node setup, not the strategy class.
2. **Adapter awareness**: if the user targets a specific venue (e.g. Binance Futures, Interactive Brokers for forex/futures/options), point them to the matching adapter under `/nautilus_trader/adapters/` and its specific config requirements.
3. **Async & precision**: be careful with timestamp units (nanoseconds vs milliseconds) when constructing data objects — mismatches are a common source of bugs.
4. **Multi-asset clarity**: NautilusTrader supports equities, FX, futures, options, and crypto — confirm which `Instrument` subtype and venue the user means before suggesting code.
5. **Build complexity**: warn users that building from source requires Rust + Cython toolchains; recommend pip-installed wheels unless they need to modify the Rust core.
6. **License compliance**: note LGPL-3.0 (or current license) implications for any commercial fork.
