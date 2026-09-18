# Governance

_To be written once `final_idea.md` is fixed._

Will cover: per-agent tool allowlists, call and runtime budgets, HITL approval points,
artifact types the system may emit, and how each maps to OWASP ASI 2026 items. Expected to
mirror the shape of the course's `day_four/config/governance/*.yaml`.

## Structure (agreed 2026-09-13, D-017)

Every rule gets three parts: the rule, where it is enforced in `app/`, and the notebook cell
that demonstrates it. A rule without a notebook assertion is not considered enforced.

| Rule | Enforced in | Demonstrated by |
|---|---|---|
| e.g. A run may not exceed N tool calls or $X | `app/policy/budget.py` | `notebooks/05_governance.ipynb` — cell "budget breach halts run" |
| e.g. Retrieved filing text may not change agent instructions | `app/security/scanner.py` + skeptic | `05_governance.ipynb` — cell "injected filing, memo unchanged" |
| e.g. Monte Carlo horizon may not exceed lookback | `app/tools/monte_carlo.py` | `05_governance.ipynb` — cell "20-year request refused" |
| e.g. No agent may call a tool outside its allowlist | `app/policy/allowlist.py` | `05_governance.ipynb` — cell "policy denial in trace" |
