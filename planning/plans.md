# Capstone plans

Three candidate plans, each scored against `capstone_requirements.md`. All three share the
same base stack (below); they differ in topology, data, and what the system answers.

Pricing snapshot (Anthropic first-party rates, 2026-06): Opus 5 $5 / $25 per 1M input / output
tokens; Sonnet 5 $2 / $10; Haiku 4.5 $1 / $5. Cached input reads are ~10 % of the input rate.
The course's maturity model names OpenRouter / LiteLLM at the MVP stage, so the provider is swappable —
the grader does not care which model answers, only that fallback and normalisation exist.

---

## Shared base stack (all plans)

| Layer | Choice | Why |
|---|---|---|
| Language / entry point | Python 3.12; one entry point `app.run(request) -> event stream` (no HTTP layer in the capstone) | D-019; FastAPI is a follow-up wrapper |
| Orchestration | **LangGraph** — supervisor graph, `interrupt()` for HITL, SQLite/Postgres checkpointer | The course's own library; checkpoints give "replay" (G5) and time-travel debugging |
| Model access | Anthropic SDK (`anthropic`) behind a thin `ModelRouter`; **LiteLLM** optional if we want provider fallback | Fallback model tested through the whole pipeline |
| Structured output | **Pydantic v2** models; tier 1 strict schema, tier 2 **Instructor** validate + retry; enums for every label | D-011 |
| Skills | `skills/*/SKILL.md` loaded into agent context by name | Component C6; mirrors `day_one/SKILL_consulting.md` |
| Tracing / evals | **Langfuse** self-hosted in docker-compose (or LangSmith free tier if compose gets heavy); **pytest** for deterministic checks; **Ragas** for retrieval quality | C7; G5 "traced, scored, replayed" |
| Security | Per-agent tool allowlists from `governance.yaml`; input/output scanner (regex + **Prompt Guard** from Hugging Face, via LlamaFirewall if it installs cleanly); **deepteam** red-team suite; path validation on any file tool; turn + cost circuit breaker | C8; G6; G7 |
| Local models | **Hugging Face Hub** — `sentence-transformers` embeddings (CPU), Prompt Guard (gated, needs `HF_TOKEN`), Chronos for Plan B | $0 per call after download; embeddings never hit a paid API |
| UI | **None.** Notebooks are the interface (input → run → memo → trace link → scores) | D-020; Part 3 accepts a notebook |
| Packaging | docker-compose (langfuse, postgres, python runner for pytest + papermill) | D-010, D-020 |
| Delivery order | **Notebooks** are the graded deliverable; nothing after them in the capstone | D-015, D-020 |
| Notebooks | `notebooks/0x_*.ipynb` importing from `app/`, run by a human at each checkpoint and by `papermill` in CI | D-004 |

Shared infrastructure cost: **$0** — everything runs locally in Docker. SEC EDGAR and Yahoo
Finance are free. Langfuse self-hosted is free.

---

## Plan A — Filings Research Desk

**Topology:** hierarchical supervisor + reflective skeptic→judge loop. Message passing between
agents; a shared "evidence board" holding retrieved passages; an event-driven approval gate on
any outbound fetch that is not EDGAR.

### High-level description
A user asks a question about a small-cap company — *"How many months of cash runway does
Longeveron have at its trailing burn rate, and how many equity raises has it done since 2023?"* The
supervisor decomposes it; a **retrieval agent** pulls the right filings and sections from
EDGAR and grounds passages with citations; a **figures agent** reads the financial-statement
tables and cross-checks them against XBRL company facts; an **analyst agent** drafts a memo;
a **skeptic agent** argues the opposite case and flags unsupported claims; a **judge** either
validates the memo (every number traceable to a tool result) or bounces it back, at most
N times, before escalating to a human. Output: a typed `ResearchMemo` with citations,
confidence, and an audit trail.

### What it answers
- "What does company X's filing say about Y, and do the numbers support it?"
- Year-over-year change in risk factors, MD&A, segment revenue, customer concentration,
  debt covenants — anything a junior equity analyst would be asked to pull from filings.
- For the grader: "Can a multi-agent system read primary sources and produce an answer whose
  every figure can be checked against a deterministic source?"

### What will be used (beyond the base stack)
- `edgartools` — EDGAR submissions, filing sections (Item 1A, Item 7, financial statements),
  XBRL company facts.
- Chunking + embeddings + a local vector store (Chroma, or pgvector in the compose Postgres).
- **Seeded Monte Carlo runway / dilution tool** over XBRL cash-flow history, with horizon and lookback caps, assumptions returned with every result (C4 — see `ideas.md`). Takes Plan A to 7 of 8. No price data (D-024).
- Golden set: ~30 (ticker, question, expected figures, expected citations) rows built by hand
  from XBRL — deterministic ground truth.

### Components and criteria
C1 ✓ C2 ✓ C5 ✓ C6 ✓ C7 ✓ C8 ✓ (C4 stretch) — **6 of 8**.
Strong on G1, G3, G4, G5, G6, G7, G8. G2 (multimodal) is covered by treating tables as a
distinct path from prose; we state plainly that audio and charts are out of scope.

### Cost
| Item | Estimate | Assumptions |
|---|---|---|
| Tokens per run | ~80–120k input, ~10–15k output | 6 agent calls, ~10 retrieved chunks each, memo + skeptic + judge |
| $ per run — all Opus 5 | ~$0.75–1.00 | no caching |
| $ per run — Opus 5 judge/analyst, Haiku 4.5 retrieval/figures | ~$0.35–0.45 | recommended mix; filing chunks cached |
| Dev + eval runs over 4 weeks | ~600–900 runs | 30-case golden set × ~15 eval sweeps + ~300 ad-hoc |
| **Project API total** | **~$250–450** (mixed) / **~$600–900** (all Opus) | |
| Other services | $0 | EDGAR is the only data source (D-024) |
| Your time | ~60–80 h | 4 build weeks; Python ramp-up included |

### What you will learn
- LangGraph supervisor graphs, `interrupt()`/`Command(resume=...)`, checkpointers.
- RAG done properly: chunking strategy, reranking, **grounding with citations**, Ragas.
- Structured-output engineering: strict schemas, Instructor retries, enum normalisation.
- Building a **defensible eval**: deterministic figure checks vs. LLM-judged prose, and
  how to set thresholds.
- Prompt-injection through *documents* (a filing containing instructions) and how a
  scanner + allowlists + skeptic stop it.
- Reading SEC filings and XBRL — domain knowledge you can reuse in StockAggregator.

### Risks
- Filing sections are long; context pruning and caching matter from day one.
- The skeptic→judge loop can burn tokens without a hard cap — cap it at 2.
- Table extraction from HTML filings is fiddly; XBRL is the fallback for figures.

---

## Plan B — Earnings Event Desk

**Topology:** event-driven fan-out (fetch agents wake on an "earnings filed" event) → shared
blackboard → single consolidation agent (message passing) → judge. The hybrid the course recommends.

### High-level description
The trigger is a company filing an 8-K with an earnings exhibit (EX-99.1). Three lightweight
**fetcher agents** run in parallel: one pulls the press-release text and the XBRL facts for
the quarter, one pulls the price/volume reaction from Yahoo Finance and a **time-series
forecast tool** (Chronos or a simple baseline) for what was "expected," one pulls prior-quarter
guidance from the previous 8-K. They post to a blackboard; a **consolidation agent** writes an
*Earnings Event Brief*: what was reported, how it compared to prior guidance and to the
forecast, how the market reacted, and what management said they expect next. A **judge**
validates figures and citations. Termination: blackboard marked complete or 2 rounds.

### What it answers
- "What happened when company X reported, and did reality match expectation?"
- Reported vs. guided vs. forecast; price reaction vs. typical move; what changed in guidance.
- For the grader: "Can an event-driven multi-agent system combine documents, structured
  data and time series into one grounded brief, with consolidation and termination logic?"

### What will be used (beyond the base stack)
- `edgartools` — 8-K exhibits, XBRL facts for the quarter.
- `yfinance` — OHLCV around the event date.
- `chronos-forecasting` (Amazon Chronos, CPU is fine for small series) **or** a
  statsmodels baseline — used as a *tool*, evaluated with MASE / coverage against actuals.
- Optional: Exa for post-earnings news (adds an API key and an injection surface — a
  feature for the security demo, but optional).
- Golden set: ~25 historical earnings events with known reported figures, prior guidance,
  and realised 1/5-day price moves.

### Components and criteria
C1 ✓ C4 ✓ C5 ✓ C6 ✓ C7 ✓ C8 ✓ (C2 stretch via XBRL tables) — **6 of 8**.
Strongest on **G2** (time series get their own handling) and G7 (event bounding is
the whole design). Slightly weaker on G1 unless the blackboard is disciplined.

### Cost
| Item | Estimate | Assumptions |
|---|---|---|
| Tokens per run | ~60–90k input, ~8–12k output | 3 parallel fetchers (small), consolidation, judge |
| $ per run — mixed (Opus 5 consolidation/judge, Haiku 4.5 fetchers) | ~$0.30–0.40 | |
| Dev + eval runs | ~600–900 | 25-case golden set |
| **Project API total** | **~$200–400** (mixed) | |
| Other services | $0–20 | Exa optional; Chronos runs on CPU |
| Your time | ~80–100 h | more moving parts: event bus, forecast tool, three data sources |

### What you will learn
- Event-driven coordination and blackboard memory in LangGraph (fan-out, `Send`, reducers).
- Time-series models as agent tools and how to eval a forecast honestly.
- Consolidation and termination logic — the failure mode the course warns about most.
- Combining three modalities (text, structured facts, price series) without a "generic path."
- Everything in Plan A's eval/security list, but the injection demo comes via news.

### Risks
- Three data sources = three flaky things; needs recorded fixtures for tests from the start.
- Forecast evals are easy to do badly; keep the forecast tool simple and the eval strict.
- Blackboard without termination logic burns tokens — hard round cap.

---

## Plan C — Compliance Flagging Committee

**Topology:** strict hierarchical pipeline with a human authority node. Message passing only.
The smallest, most governance-heavy plan.

### High-level description
Input is a draft research note or a proposed trade ticket (JSON). A **policy-retrieval agent**
pulls the relevant clauses from a policy corpus (an investment-policy statement, a restricted
list, disclosure rules, position limits — a synthetic corpus we write). A **reviewer agent**
checks the input against each clause and emits typed findings (`pass / flag / block`, with
the clause cited). A **remediation agent** proposes edits. Every `block` raises a HITL
interrupt: the human approves, edits, or rejects, and the decision is written to an immutable
audit log. Nothing executes — the output is a signed review record.

### What it answers
- "Does this note or trade comply with our policies, and which clause decides it?"
- For the grader: "Can a governed agent pipeline enforce a policy corpus with a human
  authority boundary, a full audit trail, and evals that prove seeded violations are caught?"

### What will be used (beyond the base stack)
- A hand-written policy corpus (10–20 markdown documents) — the only "data source."
- Vector store for clause retrieval; exact-match citation check.
- Golden set: ~40 inputs, half with seeded violations; the eval demands 100 % recall on
  `block`, plus precision and clause-citation accuracy.
- Optional `yfinance` to check position limits against live prices (C4 stretch).

### Components and criteria
C1 ✓ C5 ✓ C6 ✓ C7 ✓ C8 ✓ — **5 of 8**.
Strongest on **G6** and **G7**; weakest on **G2** (no multimodal input at all) and it is
less "stocks" than the others.

### Cost
| Item | Estimate | Assumptions |
|---|---|---|
| Tokens per run | ~20–40k input, ~3–5k output | short inputs, few clauses |
| $ per run — mixed | ~$0.10–0.20 | |
| Dev + eval runs | ~800–1000 | cheap runs encourage more eval sweeps |
| **Project API total** | **~$100–200** | |
| Other services | $0 | |
| Your time | ~40–60 h | least integration work; most time goes into the policy corpus and evals |

### What you will learn
- HITL mechanics end to end: `interrupt()`, approve/edit/reject, resume, audit log.
- Policy-as-retrieval and deterministic compliance evals (recall on seeded violations).
- Least privilege in practice: agents that can *only* read and *only* propose.
- Least about RAG over messy real documents and nothing about time series.

---

## Comparison

| | A — Filings Research Desk | B — Earnings Event Desk | C — Compliance Committee |
|---|---|---|---|
| Components | 6 (+1 stretch) | 6 (+1 stretch) | 5 (+1 stretch) |
| Fits the brief's first example ("document analysis") | ✓✓ | ✓ | ✓ (compliance flagging is also named) |
| G2 multimodal | tables vs prose | time series + tables + text | weak |
| Real data risk | medium | high | none (synthetic) |
| API cost (mixed models) | ~$250–450 | ~$200–400 | ~$100–200 |
| Your hours | 60–80 | 80–100 | 40–60 |
| Reuse in StockAggregator later | filings memo per ticker | earnings brief per event | low |
| Demo "wow" | memo with clickable citations; injected filing blocked | live event → brief; forecast vs. actual | a blocked trade with the clause that blocked it |

### Recommendation
**Plan A**, with Plan C's HITL approve/edit/reject pattern applied to its approval gate, and
the seeded Monte Carlo tool over XBRL cash flows as the C4 stretch (no price data, D-024). It is the plan the brief describes first,
it has deterministic ground truth (XBRL) for the eval, and it teaches the two things you said
you're least sure about — what a research agent reads, and how to keep it honest.

If the four build weeks look tight after the Part 1 review, Plan C is the safe fallback:
half the hours, all the governance vocabulary, and a clean 100 %-recall eval story.
