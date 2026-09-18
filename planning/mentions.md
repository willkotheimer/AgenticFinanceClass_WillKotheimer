# Mentions triage

Everything the course mentions, sorted by what we do with it. The rubric requires **three of
eight components** and grades qualities, not tools; naming a non-goal with a reason scores as
well as building it. Update as she mentions more.

Buckets: **Build** (core of Plan A) · **Cheap add** (< half a day, earns its place) ·
**Name only** (appears in the design doc as a considered non-goal) · **Skip** (not mentioned).

## Build

| Mention | Role in Plan A | Why |
|---|---|---|
| LangGraph | supervisor graph, `interrupt()`, checkpointer | C5, G1, G5 replay |
| LangChain | components only — message types, `@tool`, text splitters, loaders, vector-store adapters (LangGraph depends on it) | parts bin, not architecture; a LangChain *chain* is the "chain of calls" G1 rejects |
| Pydantic v2 | every state object and output DTO | D-016; tier-1 schema |
| Instructor | tier-2 validate + retry | D-011 |
| Langfuse **or** LangSmith | traces, spans, eval datasets | C7, G5 |
| pytest | deterministic checks, fixture replay | D-005 |
| Prompt Guard (HF) + regex scanner | input/output injection scan | C8 |
| Allowlists, budgets, circuit breaker | `governance.yaml` + policy module | G6, G7 |
| Checkpointing | LangGraph checkpointer (SQLite/Postgres) | crash recovery + replay |
| HITL approve/edit/reject | approval gate on non-EDGAR fetches; memo sign-off | orchestrated autonomy: approve / edit / reject |
| SKILL.md | at least one skill file (memo-writing, filing-reading) | C6 |
| Docker / compose | how the grader runs it | G8 |
| `edgartools` | the only data tool; EDGAR is the sole outbound source (D-024) | C1, C2, C4 via XBRL |
| HF `sentence-transformers` | local embeddings | $0 RAG |
| Structured-output enum normalisation | small stable enums for labels | defeats label drift across models |
| Fallback model | second model tested through the whole pipeline | via LiteLLM or a second SDK client |

## Cheap add

| Mention | Where | Why it earns its place |
|---|---|---|
| **Pandera** | schemas on XBRL and price DataFrames at the tool boundary | the DTO for tables; ~20 lines; same contract idea as D-017 |
| deepteam | red-team suite run as one eval | C8 evidence beyond one hand-made injection |
| Seeded Monte Carlo tool | runway / dilution with horizon caps | C4 → 7 of 8; deterministic eval |
| Ragas | retrieval-quality metrics | C7 for the RAG stage |
| Context pruning | token-aware truncation of history | filings are long |
| Optuna over retrieval params | chunk size / top-k vs recall@k | stretch; $0; G4/G5 |

## Name only (design-doc non-goals with a reason)

| Mention | Reason we don't build it |
|---|---|
| E2B, monty, Firecracker, gVisor | we never execute model-generated code; sandboxing is moot — state this as a "may not do" |
| MCP | protocol boundary adds no capability in a single process; plain typed tools with the same allowlist |
| A2A | no cross-vendor agents; governance is in our own policy layer |
| Kubernetes, Ray Serve, vLLM / TGI / SGLang | Scale stage; hosted inference, no self-hosting (D-014) |
| Streamlit | React SPA instead (D-015), same zero-trust boundary |
| Chronos / PatchTST | Plan B only; Plan A's C4 is Monte Carlo, not forecasting |
| Exa / Composio | no web search; EDGAR is the only outbound source (smaller injection surface) |
| RL (ART, RULER, GRPO) | no fine-tuning; evals are for measurement, not training |
| Audio / video models (Qwen2-Audio, Voxtral) | no audio modality; stated out of scope |
| OpenRouter / LiteLLM | only if the fallback model is on another provider |
| Ray Tune | Optuna does the same job and I already know it |

## Skip

Nothing yet — everything above is at least worth a sentence in the design doc.
