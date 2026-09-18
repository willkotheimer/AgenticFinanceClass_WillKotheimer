# Test strategy

How we know the system works and is not just "doing things." Written before any code exists.
All expected values come from EDGAR, never from an agent run.

## Principle

Every score has a floor (a baseline) and a ceiling (an oracle the agent cannot fake). A score
without both is a claim, not a measurement.

## Oracles — what a run is compared against

| Oracle | Checks | Fakeable | Source |
|---|---|---|---|
| XBRL company facts | every figure in the memo | no | `data.sec.gov/api/xbrl/companyfacts` |
| Submissions index | filing events: S-1 count, 8-K Items 3.01 / 5.03, form types | no | `data.sec.gov/submissions` |
| Hand-verified passages | each citation points at text that supports the claim | no | pinned filing text, string containment |
| Fixed seed | Monte Carlo tool output | no | byte-identical |
| LLM judge | prose quality, reasoning coherence | yes | used last, never alone, always beside a deterministic score |

## Test types

### 1. Deterministic (pytest, no LLM)
- Tool wrappers against recorded fixtures: EDGAR fetch, XBRL parse, table extraction, chain
  of citations.
- Policy: allowlist denial, budget breach, circuit breaker trip, path validation.
- Schemas: every DTO round-trips; every enum rejects unknown labels.
- Monte Carlo: fixed seed → identical output; horizon > lookback → refusal.

### 2. Golden-set evals (LLM in the loop, scored)
~30 cases over the roster, 3–5 per company. Each case: ticker, question, expected figures,
expected citation passages, expected decision enums.
- **Figure match** — exact or within declared tolerance.
- **Citation coverage** — share of claims with a citation that contains the claim.
- **Decision enums** — going-concern flag, confidence, judge verdict.
- **Memo quality** — LLM judge, reported beside the deterministic scores, never as the headline.

### 3. Baselines (the floor)
- **No-retrieval** — same question, no tools. Must be clearly beaten; if not, retrieval adds nothing.
- **Trivial** — "always answer the most common value" per enum.
- **Single-agent ablation** — analyst alone vs. analyst + skeptic + judge. If the extra agents
  do not move figure match or citation coverage, they are not justified.

### 4. Negative controls (the agent must fail correctly)
- **Wrong filings** — Company A's filings, question about Company B → refusal or "not found."
- **Unanswerable** — figure not in the source → "not in source," no invented number.
  The most important test in the set; hallucination is invisible without it.
- **Poisoned filing** — injected instructions in retrieved text → memo unchanged, event logged.
- **Extrapolation trap** — 20-year Monte Carlo request → bounded refusal.

### 5. Metamorphic (same truth, different phrasing)
- Rephrased question → identical figures.
- Units changed (thousands vs. millions) → correct conversion.

### 6. Discrimination (contrast pairs from the roster)
Pairs chosen by filing facts, never by price: LGVN vs. RXRX (going concern vs. well-funded),
MNTS vs. LUNR (messy vs. clean space), KDK vs. BFLY (3 vs. 6 annual reports). Memos must reach
different, correct conclusions. Identical cautious memos for every company = "doing things."

### 7. Stability (repeat runs)
~10 cases × 5 runs × {primary, fallback, one open model}. Decision enums must not flip.
Isolate the judge step on a cached memo to keep cost ≈ $5.

## Process rails

- **Red before green.** A failing test exists before any `app/` code for that feature.
- **Cases before agents.** Golden cases are written and verified against EDGAR before the
  agent that answers them exists.
- **One ticker per run.** The roster lives only in the eval, run as sweeps at milestones.
- **Sign-off per checkpoint.** Each notebook checkpoint is run by me and recorded in
  `questions.md` before the next milestone starts.
- **Baseline beside every score.** No eval reports a number without its floor.
- **Failures are reported.** Part 3 asks for what failed; the eval report has a section for it.

## What "working" means, in one sentence

Figure match and citation coverage clearly above the no-retrieval baseline, the ablation
showing the skeptic and judge earn their cost, zero invented figures on the unanswerable set,
and decision enums stable across runs.
