# Ideas log

Running log of capstone ideas. Newest at the bottom. When an idea is chosen it moves to
`final_idea.md`; the rest are summarised in `alternate_followup_ideas.md`.

Course constraints (from the professor): multi-agent orchestration, evals, security. Any topic.
Ours: stocks; Python + FastAPI + Docker (no .NET); emphasis on *research* agents (e.g. reading
SEC filings) over trading automation.

---

## 2026-09-13 — first brainstorm

### 1. Signal Desk — "second opinion" on a market signal
A supervisor dispatches `news`, `technicals`, `fundamentals`, and `skeptic` agents; a `judge`
merges their findings into a Pydantic-typed *Signal Brief* where every number must trace back
to a tool result.
- Evals: golden set of historical signals with known 5/20-day outcomes; LLM-as-judge on brief
  quality; citation check (any uncited number fails).
- Security: prompt injection via a fetched news article; per-agent tool allowlists;
  `governance.yaml` call budgets; input/output scanning (LlamaFirewall in the course).
- Original input was a StockAggregator "hidden divergence" signal; now decoupled (see D-002).

### 2. Paper-trading investment committee
Proposer → risk → compliance → HITL approval → paper-trade tool. Strongest *excessive agency*
story (prove the agent cannot reach a real broker). Bigger scope; lots of prompt tuning.
Ties to `StockAggregator.PaperTrading` later.

### 3. "Why did it move?" event triage
Triggered by a >3 % move; explains it from news, sector peers, correlation breakdown.
Attribution is fuzzy so evals are hard to make deterministic.

### 4. Filings analyst (10-K / 10-Q / 8-K) with RAG
Closest to the professor's emphasis. Agents read SEC filings, extract risk factors, MD&A
changes year over year, and produce a cited research memo. Evals via Ragas / synthetic
Q&A sets (Day 1 notebooks). Security demo: document-borne injection (hidden text in a filing
or an attached PDF).

### 5. NL → screener
Natural language to constrained data queries over quotes. Too small alone; good as one tool
inside #1 or #4.

---

## 2026-09-13 — what can a research agent actually read? (data sources)

All free unless noted. Everything below has a Python client or is plain HTTP.

| Source | What it gives | Access | Notes |
|---|---|---|---|
| SEC EDGAR submissions | List of every filing for a company | `https://data.sec.gov/submissions/CIK##########.json` | Needs a descriptive `User-Agent` header; max 10 req/s |
| SEC EDGAR full-text search | Search filing text since 2001 | `https://efts.sec.gov/LATEST/search-index?q=...` | Good for "find 8-Ks mentioning X" |
| SEC XBRL company facts | Structured financials (revenue, EPS, …) per period | `https://data.sec.gov/api/xbrl/companyfacts/CIK##########.json` | Deterministic numbers → great for evals |
| SEC XBRL frames | One concept across all companies for a period | `https://data.sec.gov/api/xbrl/frames/...` | Peer comparisons |
| Filing documents | 10-K, 10-Q, 8-K HTML/PDF | `https://www.sec.gov/Archives/edgar/data/CIK/ACCESSION/` | Chunk + embed for RAG; `edgartools` or `sec-edgar-downloader` wrap this |
| Yahoo Finance chart API | OHLCV history, quotes | `https://query1.finance.yahoo.com/v8/finance/chart/{symbol}` | Same endpoint StockAggregator uses; `yfinance` wraps it |
| Exa | Semantic news/web search | API key (paid tier, free credits) | Used in the course's news notebooks |
| Financial PhraseBank | Labelled sentiment sentences | HuggingFace dataset | Used in a Day 1 notebook; handy for a sentiment sub-eval |
| Earnings-call transcripts | — | Mostly paid | Skip unless a free source appears; 8-K earnings exhibits (EX-99.1) are a free substitute |

Python libs worth evaluating: `edgartools` (highest level, parses XBRL + filing sections),
`sec-edgar-downloader` (just downloads), `yfinance`, `langgraph`, `langfuse`, `ragas`.

---

## 2026-09-13 — process ideas

### Jupyter notebook checkpoints
Some milestones should be demonstrated in a notebook, run by a human, not just claimed by an
agent. Rationale: keeps agent (Claude) redirection honest and mirrors the course's notebook
format, which the professor will recognise.
- Notebooks *import from* `app/`; no business logic lives in a notebook.
- Each notebook is also executed headless in CI (`papermill` or `nbconvert --execute`) so it
  doubles as a smoke test.
- Candidate checkpoints: (1) data source reachable + parsed, (2) single agent end-to-end,
  (3) multi-agent graph trace in Langfuse, (4) eval suite run with scores, (5) injection
  attack blocked.

### Evals + deterministic TDD
Two layers, both run in CI:
- **Deterministic (pytest):** tool wrappers, parsers, schema validation, policy/allowlist
  enforcement, budget accounting. No LLM calls; use recorded fixtures.
- **Evals (LLM in the loop):** golden datasets, LLM-as-judge, citation checks. Scored, with
  thresholds that fail the build if they regress.

---

## 2026-09-13 — re-scored against the capstone spec

The course brief defines eight components (C1–C8) and eight grading criteria (G1–G8); see
`capstone_requirements.md`. Which components each idea hits *naturally* (✓) or with effort (~):

| Idea | C1 retrieval | C2 tables | C3 audio | C4 market data | C5 tools | C6 SKILL.md | C7 evals | C8 security | Natural count |
|---|---|---|---|---|---|---|---|---|---|
| 1 Signal Desk | ~ | | | ✓ | ✓ | ✓ | ✓ | ✓ | 5 |
| 2 Paper-trading committee | | | | ✓ | ✓ | ~ | ✓ | ✓ | 4 |
| 3 Why did it move | ~ | | | ✓ | ✓ | ~ | ~ | ✓ | 3 |
| 4 Filings analyst | ✓ | ✓ | ~ | | ✓ | ✓ | ✓ | ✓ | 6 |
| 5 NL screener | | | | ✓ | ✓ | | ✓ | ✓ | 4 |

Idea #4 scores highest and is the one the brief names first ("document analysis"). Merging
#1's *skeptic + judge + citation check* into #4 gives the strongest coverage of G1 (harness, not
a chain), G4 (bounded, classified failures) and G5 (score you can defend).

### Working title: **Filings Research Desk**
A governed multi-agent workflow that, given a ticker and a question, retrieves the relevant
SEC filings, extracts structured figures from the financial-statement tables (C2), grounds
every claim in a cited passage (C1), and produces a validated research memo — with a
skeptic agent, a judge, an approval gate before any outbound web fetch, and a scored eval
suite. Market data (C4) appears as one governed tool, not the centre of the system.

Topology (using the course vocabulary): **hierarchical** supervisor for control and
auditability (fits "regulated environments"), with a **reflective** skeptic→judge step for
precision. **Message passing** between agents (explicit contracts, inspectable logs), a
**shared memory** research board only for the retrieved evidence, and an **event-driven**
governance gate for tool approvals. Termination: declared max turns, cost budget, and a
"memo validated or escalated to human" exit.

Threats to name in the design doc (via the six-question decision path): T2 tool misuse
(filing text injecting instructions), T5 cascading hallucination (skeptic + citation check),
T4 resource overload (budgets), T8 repudiation (immutable trace), T6 goal manipulation
(input scanning), T11 RCE only if we add code execution (then E2B/Docker sandbox).

### Part 1 proposal outline (what `final_idea.md` becomes)
1. Intro — what the workflow is, who uses it, what question it answers (half a page).
2. Workflow diagram — supervisor, agents, tools, approval gate, memory, exits.
3. Component mapping — a table of C1/C2/C5/C6/C7/C8 → concrete piece of the system.
4. Roles and decision points — which agent owns which decision; where a human is asked.
5. Context, tools, memory — what each agent is allowed to see and call.
6. Human checkpoints — approval gate on web fetches; memo sign-off; notebook checkpoints.
7. What we will *not* build — no trade execution, no audio, no live broker connectivity.

### Design-doc items the rubric will look for (Part 2)
- "May and may not do" list → `governance.md`.
- Per-agent allowlists, budgets, circuit breaker, checkpointing, context pruning.
- Structured-output tiers 1+2 with enum normalisation; raw outputs logged.
- Fallback model tested through the whole pipeline.
- Failure classification for every tool (timeout / bad-args / empty / policy-denied) and the
  recovery path for each.
- Redaction rules written before tracing is switched on.
- Eval scorecard: deterministic checks (XBRL figure match, citation coverage, schema
  validity, policy denials) plus LLM-judged memo quality, with thresholds.

---

## 2026-09-13 — eval roster: small caps, verified on EDGAR

Why small caps beat mega caps for this capstone (put this argument in the Part 1 proposal):
with NVDA or MSFT the model can answer from parametric memory, so the eval cannot tell a
*grounded* answer from a *memorised* one. With BFLY or LGVN the model knows almost nothing —
if the figure is right, it came from retrieval. Small caps make the citation check test the
thing it claims to test. They also carry the questions worth asking: going-concern language,
cash runway (cash ÷ burn), dilution via S-1s, reverse splits (8-K Item 5.03), Nasdaq
listing-deficiency notices (8-K Item 3.01), customer concentration, related-party deals.

| Ticker | Company | Sector | Files | Why it is in |
|---|---|---|---|---|
| BFLY | Butterfly Network | Medtech (handheld ultrasound) | 10-K | clean baseline; SPAC-era S-1s |
| KDK | Kodiak AI | Autonomous trucking | 10-K (only 3) | young filer; YoY limited to two comparisons |
| POET | POET Technologies | Photonics | **20-F / 6-K** | Canadian FPI; IFRS XBRL tags; **add last** |
| LGVN | Longeveron | Cell-therapy biotech | 10-K | 11 S-1s, 4 charter amendments, 5 listing-deficiency 8-Ks — the risk minefield |
| RXRX | Recursion Pharmaceuticals | Platform biotech | 10-K | well-funded contrast to LGVN; cash-runway questions |
| LUNR | Intuitive Machines | Space | 10-K | contract revenue; 5 S-1s — dilution questions |
| BARK | Bark, Inc. | Consumer retail | 10-K | real COGS/inventory — different statement shape |
| SOUN | SoundHound AI | AI software | 10-K | recurring revenue; share-based comp |
| MNTS | Momentus | Space (messy) | 10-K | 14 S-1s, going-concern — the "something fails" case |

Question templates that work across the roster (each has a deterministic XBRL or 8-K answer):
- Cash and equivalents at FY end, and months of runway at the trailing operating cash burn.
- Shares outstanding YoY and the number of equity raises (S-1 / 424B) in the period.
- Does the latest 10-K contain going-concern language? Cite the paragraph.
- Any reverse split or listing-deficiency notice in the last 24 months? Cite the 8-K item.
- Largest customer concentration disclosed, and how it changed vs. prior year.
- Top three risk factors added or removed vs. the prior 10-K (LLM-judged, citation-checked).

Verification method: `https://www.sec.gov/files/company_tickers.json` for CIKs, then
`data.sec.gov/submissions/CIK##########.json` for form counts and 8-K item codes.

---

## 2026-09-13 — seeded Monte Carlo as a governed C4 tool

Monte Carlo is stochastic by construction but **reproducible given a seed** — same seed and
inputs, identical output. That makes it a good agent *tool*: the LLM decides what to ask, the
tool computes the number, pytest can assert byte-identical results.

The danger is not the sampling, it is the assumptions: compound four years of NVDA growth for
twenty and $60k becomes $1M+. So the tool must be governed, not free-form:
- Horizon cap (runway sims to 24–36 months; never multi-decade).
- Lookback ≥ horizon — cannot project further than observed.
- Result carries its assumptions: `{p10, p50, p90, seed, lookback, horizon, distribution}`;
  the memo must cite all of them, not just the P50.
- Skeptic flags any extrapolation the assumptions do not support.
- Pre-built tool with declared arguments; **no model-generated simulation code** (avoids the
  T11 RCE surface). Record that as a "may not do" in `governance.md`.

Candidate simulations for the small-cap roster:
- Cash runway — burn sampled from the last 8 quarters of operating cash flow (XBRL);
  months until cash < 0, P10/P50/P90.
- Dilution — share count in 24 months given historical raise cadence (S-1 / 424B counts).
- Price paths (GBM from `yfinance` realised vol) — secondary; least defensible.

Eval cases this adds: (a) fixed-seed determinism; (b) memo quotes the tool's P50 exactly;
(c) the "extrapolation trap" — ask for a 20-year projection, correct behaviour is a bounded
refusal. Adding this makes Plan A **7 of 8** components.

---

## 2026-09-13 — Hugging Face

An HF account is needed (and lowers cost — everything below runs locally after download):

| Use | Model / dataset | Gated? |
|---|---|---|
| Local embeddings for RAG (no embedding API cost) | `sentence-transformers` / BGE family | no |
| Prompt-injection classifier for the C8 scanner | Meta Prompt Guard (used by LlamaFirewall) | **yes** — accept licence, set `HF_TOKEN` |
| Forecast tool (Plan B / C4) | Amazon Chronos, `chronos-t5-small` on CPU | no |
| Sentiment sub-eval | Financial PhraseBank via `datasets` | no |
| Open fallback model (optional) | any instruct model via HF Inference or local | free tier rate-limited |

---

## 2026-09-13 — stretch: tune the harness with Optuna (or Ray Tune)

The course's maturity model puts Ray at the Scale stage (Ray Serve, self-hosted inference — out of scope). The
Day 1 `ray_tune.ipynb` is Ray *Tune*, i.e. HPO — same job as Optuna, which I know from
CatBoost. The useful application here is **harness tuning, not model tuning**:

- Knobs: chunk size, chunk overlap, top-k, reranker threshold, (maybe) query rewriting on/off.
- Objective: retrieval recall@k — "does the gold passage land in the top-k?" — which is
  **deterministic and makes zero LLM calls**, so a 50-trial study costs CPU only.
- Split the golden set (train / validation) or the 30 cases will be overfit.
- Do **not** put LLM-scored metrics (memo quality, citation coverage) in the objective:
  ~20 trials × a full LLM sweep ≈ $250 for a number that is too noisy to trust.
- Deliverable: one notebook checkpoint + a line in the design doc: "chunk_size=N chosen by
  Optuna over retrieval recall on the validation split (study attached)."

Stretch only; scores on G4/G5 and shows an eval being used as an optimisation objective.

---

## 2026-09-13 — eval: decision-stability bake-off

The course showed a bake-off: several models, same input, repeated runs. All returned valid
JSON; one flipped a priority label between runs. Valid JSON does not imply stable decisions.

Replicate for Plan A as one eval:
- ~10 golden cases × 5 runs × {primary, fallback} model.
- Measure decisions only — the enums: judge verdict (`validated / bounced / escalated`),
  memo confidence, going-concern flag (`yes / no / unclear`). Ignore prose.
- Report a consistency matrix per model per label ("consistent" / "flips"), plus structured-
  output tier reached and latency.
- Isolate the decision step: re-run the judge on a cached memo, not the whole graph.
  Cost ≈ 10 × 5 × 2 × $0.05 ≈ **$5** (full-graph version ≈ $40).

What it proves: the fallback model works *through the pipeline*, and enum normalisation
holds. A flip somewhere is expected — report it and the fix.

Model sourcing for the bake-off: `vendor/model` IDs (`openai/…`, `qwen/…`, `minimax/…`) are
**OpenRouter** names — hosted, one key, no downloads. Do not download bake-off models from
Hugging Face; the large ones will not run locally. Lineup of **three**: primary and fallback
Claude models via the Anthropic SDK, plus one open-weight model (Qwen or Llama) via OpenRouter
as the likely label-flipper. A fourth is optional. `ModelRouter` hides the endpoint, so the
lineup is config, not code. Local HF downloads stay limited to embeddings, Prompt Guard, and
Chronos-small.

---

## 2026-09-13 — domain switch: P&C insurers (D-021, reverted by D-022 — kept as a follow-up)

Professor allows own-domain projects. Insurance is finance-adjacent and insurers are SEC
filers, so Plan A's harness is unchanged; only the roster and questions change.

| Ticker | Company | Line | Why it is in |
|---|---|---|---|
| ROOT | Root | insurtech auto | my lane; 3 charter amendments |
| LMND | Lemonade | insurtech multi-line | growth-over-profit; reserve story |
| HIPO | Hippo | insurtech home | 2 listing-deficiency notices — the minefield |
| KINS | Kingstone | NY homeowners | tiny; 18 years of 10-Ks; 3 deficiency notices |
| HCI | HCI Group | Florida homeowners | cat exposure; 6 charter amendments |
| HRTG | Heritage | Florida homeowners | second view of the same cat risk |
| UFCS | United Fire | commercial P&C | old-line clean baseline |
| KNSL | Kinsale | E&S specialty | the well-run contrast |
| NODK | NI Holdings | regional, crop + farm | small, different book |

What insurer 10-Ks add that generic small caps do not:
- **Loss development triangles** (ASU 2015-09 disclosure) — a table with a textbook
  deterministic calculation (chain-ladder). C2 done properly.
- Combined ratio, prior-year reserve development, reinsurance recoverables, cat losses —
  deterministic from XBRL or the statement tables.
- **Monte Carlo with a real actuarial use**: bootstrap the triangle for reserve variability
  (replaces the cash-runway sim as the C4 tool).

Insurance question templates (deterministic answers):
- Combined ratio for FY, and its loss / expense split; change vs. prior year.
- Prior-year reserve development (favourable / adverse) for the last three years, cited.
- Net ultimate loss for accident year N as reported in successive triangles — did it drift?
- Reinsurance recoverables as a share of reserves; largest reinsurer concentration.
- Cat losses for the year and the named events.
- Any listing-deficiency notice or charter amendment in 24 months (8-K Items 3.01 / 5.03).
- Top risk factors added or removed vs. prior 10-K (LLM-judged, citation-checked).

The generic small-cap roster (BFLY, KDK, POET, LGVN, RXRX, LUNR, BARK, SOUN, MNTS) moves to
`alternate_followup_ideas.md` — same harness, different roster and templates, for later.
