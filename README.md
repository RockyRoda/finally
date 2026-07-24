# FinAlly — AI Trading Workstation

FinAlly Capstone Project - LLM driven Trader Workstation for Simulated Trading

A visually stunning AI-powered trading workstation that streams live market data, simulates portfolio trading, and integrates an LLM chat assistant that can analyze positions and execute trades via natural language.

Built entirely by coding agents as a capstone project for an agentic AI coding course. Full spec: [`planning/PLAN.md`](planning/PLAN.md).

## Status

**In progress.** Only the backend market data subsystem is built so far — see [`planning/MARKET_DATA_SUMMARY.md`](planning/MARKET_DATA_SUMMARY.md). Portfolio, watchlist, chat/LLM, frontend, and Docker packaging are not yet implemented.

## Planned Features

- **Live price streaming** via SSE with green/red flash animations
- **Simulated portfolio** — $10k virtual cash, market orders, instant fills
- **Portfolio visualizations** — heatmap (treemap), P&L chart, positions table
- **AI chat assistant** — analyzes holdings, suggests and auto-executes trades
- **Watchlist management** — track tickers manually or via AI
- **Dark terminal aesthetic** — Bloomberg-inspired, data-dense layout

## Architecture (target)

Single Docker container serving everything on port 8000:

- **Frontend**: Next.js (static export) with TypeScript and Tailwind CSS
- **Backend**: FastAPI (Python/uv) with SSE streaming
- **Database**: SQLite with lazy initialization
- **AI**: LiteLLM → OpenRouter (Cerebras inference) with structured outputs
- **Market data**: Built-in GBM simulator (default) or Massive API (optional) — done

## Try the Market Data Demo

The only runnable piece today is the backend market data module:

```bash
cd backend
uv sync --extra dev
uv run market_data_demo.py
```

Displays a live-updating terminal dashboard with 10 tickers, sparklines, and simulated price moves.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENROUTER_API_KEY` | For chat (not yet built) | OpenRouter API key for AI chat |
| `MASSIVE_API_KEY` | No | Massive (Polygon.io) key for real market data; omit to use simulator |
| `LLM_MOCK` | No | Set `true` for deterministic mock LLM responses (testing) |

## Project Structure

```
finally/
├── backend/     # FastAPI uv project — market data module complete
├── planning/    # Project documentation and agent contracts
├── test/        # Playwright E2E tests (planned)
└── scripts/     # Start/stop helpers (planned)
```

`frontend/`, `Dockerfile`, and `.env.example` are planned but not yet created.

## License

See [LICENSE](LICENSE).
