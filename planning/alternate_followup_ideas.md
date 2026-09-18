# Alternate and follow-up ideas

Ideas not chosen for the capstone, kept for later. Full write-ups are in `ideas.md`.

## Follow-ups after the class

- **FastAPI wrapper + React UI** — `POST /runs` + SSE around `app.run()`, so StockAggregator or
  a React SPA can call the agents. Dropped from the capstone by D-019 / D-020.

- **StockAggregator integration** — a `MarketDataProvider` implementation that reads
  StockAggregator's read-only API, and a card on its Signals page that renders the agent's
  research memo. (See D-002: out of scope for the capstone.)
- **.NET agentic-framework port** — rebuild the same graph in .NET, keeping the memo schema as
  the contract so both versions produce identical JSON.
- **Paper-trading investment committee** (idea #2) — proposer / risk / compliance / HITL
  approval on top of `StockAggregator.PaperTrading`.

- **P&C insurer roster** — ROOT, LMND, HIPO, KINS, HCI, HRTG, UFCS, KNSL, NODK with the
  insurance templates in `ideas.md` (combined ratio, reserve development, loss triangles,
  chain-ladder, bootstrap reserve variability). Same harness; swap the roster, templates, and
  the Monte Carlo tool. Roster, filing links, and XBRL availability in `insurance_companies.md`;
  primer in `primer_pc_insurance.md`. Chosen then reverted (D-021 → D-022):
  worth doing once I can judge the agent's answers myself.

- **Price and news layer** — `yfinance` price context, news via Exa, and the "news says good,
  price says bad" correlation. Dropped by D-024; revisit in a calmer regime, likely as part of
  the earnings-reaction desk (Plan B).

## Alternates considered

- **"Why did it move?" event triage** (idea #3) — attribution evals too fuzzy for a capstone.
- **NL → screener** (idea #5) — too small alone; may become a single tool in the final project.
