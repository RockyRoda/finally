# Review of `planning/PLAN.md`

## Overall assessment

The plan presents a coherent, appropriately scoped capstone with a clear product vision, sensible technology choices, and useful ownership boundaries. The simulator-first approach, single-container deployment, and explicit trade constraints should keep the project demoable.

The plan is not yet a complete implementation contract, however. The largest remaining risk is that frontend, backend, and test agents can each make reasonable but incompatible assumptions. Before parallel implementation continues, the plan should define exact API payloads, real-time data semantics, and multi-action failure behavior.

## High-priority issues

### 1. Define complete API request and response contracts

The endpoint table gives paths and broad descriptions but not response shapes. Specify JSON schemas or examples for every endpoint, including:

- money and price representation, rounding, and nullability;
- timestamps and timezone (prefer UTC ISO 8601 with `Z`);
- ordering and pagination/limits for history and chat messages;
- whether `/api/portfolio` embeds current prices and how missing/stale prices appear;
- success bodies for trade and watchlist mutations;
- a consistent error envelope such as `{code, message, details}`;
- the chat response shape after actions execute, including per-action success or failure.

Also decide whether price/money fields are JSON numbers or decimal strings. SQLite `REAL` and binary floating-point can create visible cash and P&L artifacts unless calculation and rounding rules are explicit.

### 2. Resolve the market-data/UI mismatch

The watchlist requires “daily change %,” but the market model only defines the latest and previous tick. Previous-tick change is not daily change. Add a session reference/open or rename the UI value to tick change.

The larger chart and sparklines are accumulated only after page load, so they begin empty after every refresh. State this clearly as an accepted MVP behavior or add a price-history endpoint/buffer. Define maximum retained points and downsampling so long-running tabs do not grow without bound.

### 3. Do not accept invalid Massive tickers silently

The documented source divergence allows an invalid ticker to remain on the watchlist forever without a price. That conflicts with the claims that watched tickers have live prices and are tradable. It also leaves the UI and AI in an undefined state. `POST /api/watchlist` should validate availability, time out with a clear error, or represent a pending/unpriced ticker explicitly. The same rule must apply to AI watchlist changes.

### 4. Specify atomicity and ordering for AI actions

“Watchlist changes first, then trades” is insufficient for responses containing several actions. Define:

- whether actions are best-effort or all-or-nothing;
- whether later actions continue after one fails;
- behavior for add/remove conflicts, duplicate actions, and multiple trades in one response;
- whether trade validation uses balances and positions updated by earlier actions;
- exactly what is stored and returned for requested versus executed actions.

A best-effort ordered action-result list is likely easiest to explain and test. Persist the user message even when the provider call fails, and specify whether failed assistant calls/actions are persisted.

### 5. Tighten trade consistency and concurrency rules

The transaction guarantee is good, but the plan should also cover concurrent manual/AI trade requests, SQLite locking (`busy_timeout`, WAL or an application-level write lock), and the exact price snapshot used for execution. Define stale/missing-price rejection and include the executed price in the mutation response. Add database constraints for positive quantities, valid sides, nonnegative balances, and unique ticker normalization where practical.

### 6. Clarify lifecycle ownership

Database initialization is described both as occurring “on startup (or first request)” and as lazy first-request initialization. Pick one; FastAPI lifespan startup is more deterministic. The same lifespan should start market data and snapshot tasks only after the watchlist is loaded, and stop them cleanly.

Specify how runtime watchlist changes reach the market source and how failures are rolled back. Also define snapshot retention and whether the 30-second task skips, records, or marks valuations when any held price is unavailable.

## Medium-priority issues

### 7. Make SSE behavior a testable protocol

Document the event name and data envelope, heartbeat/comment cadence, cache-control/proxy headers, initial snapshot behavior, and reconnect semantics. A newly connected client must receive current prices even when the cache version does not subsequently change. Consider event IDs if missed updates matter; otherwise explicitly state that clients receive latest state rather than every tick.

### 8. Define ticker normalization and watchlist limits

State that tickers are trimmed and uppercased, define allowed characters/length, and decide whether symbols such as `BRK.B` are supported. Add a maximum watchlist size; simulator correlation work, Massive API use, SSE payload size, and chart memory otherwise have no bound.

### 9. Reconcile configuration statements

`OPENROUTER_API_KEY` is labeled required, while the behavior section says the application works without it. Label it “optional; required only for live chat.” The README currently repeats the “required” interpretation.

The phrase “backend reads `.env` from the project root” is ambiguous in the container, where Docker `--env-file` injects variables rather than mounting the file. Prefer process environment as the contract, with local `.env` loading only as a development convenience.

The instruction to use a named coding-agent skill is not a runtime architecture requirement and may not be available to every agent. Put concrete provider/model/configuration requirements in the plan and agent-specific workflow instructions elsewhere. Avoid hard-coding a model without an environment override and defined fallback/error behavior.

### 10. Complete the chat retrieval contract

There is no `GET /api/chat` endpoint even though the UI implies conversation history and messages are persisted. Either add one or state that chat history intentionally does not survive a page refresh. Define request length, history ordering, provider timeout/retry policy, structured-output validation retries, and a stable unavailable/error response.

### 11. Define static-serving and Docker details

Specify the frontend export directory, where it is copied in the runtime image, how `/` and asset paths are served, and whether unknown non-API routes fall back to `index.html`. Use `npm ci` with a committed lockfile rather than `npm install` for reproducible builds. Clarify whether bind mounts or named volumes are the supported default; the plan and README show different approaches.

### 12. Add observable health/readiness semantics

Define whether `/api/health` only reports that the process is alive or also checks database initialization, background-task status, and market-data freshness. Separate liveness/readiness if Docker depends on it. Logs should make source selection, provider failures, SSE disconnects, trade IDs, and background-task failures diagnosable without exposing API keys or full prompts.

## Testing and acceptance criteria

The listed scenarios are a good start, but several need deterministic assertions rather than visual descriptions. Add tests for:

- simultaneous trades and duplicate submissions;
- missing/stale prices and invalid Massive symbols;
- SSE initial state, heartbeat, reconnect, and cleanup after disconnect;
- watchlist removal racing with a trade;
- multi-action chat responses with partial failures;
- malformed/timeout/provider-error LLM responses;
- database persistence across container restart;
- empty portfolio states and long-running chart point limits;
- responsive behavior at named viewport widths;
- keyboard navigation, focus visibility, accessible labels, and color-independent gain/loss indicators.

Give each headline user experience a measurable acceptance criterion—for example, a connection-status transition timing, maximum supported watchlist size, and expected behavior when chat is unavailable.

## Documentation quality

The plan also mixes product requirements, completed implementation status, agent workflow instructions, and rationale. Consider keeping `PLAN.md` as the authoritative product/API contract while completed-component notes remain in `MARKET_DATA_SUMMARY.md` and agent-specific instructions live in the relevant guidance files.

## Recommended next revision

Before assigning the remaining components, add four compact appendices:

1. OpenAPI-style request/response examples and a common error schema.
2. SSE event examples and connection lifecycle rules.
3. State-transition rules for manual trades, watchlist mutations, and ordered AI actions.
4. A short acceptance matrix mapping each user-visible feature to deterministic tests.

With those additions—and resolution of invalid ticker, daily-change, and action-atomicity semantics—the plan will be strong enough to serve as a reliable cross-agent implementation contract.
