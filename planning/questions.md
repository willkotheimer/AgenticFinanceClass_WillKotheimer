# Questions log

Dated record of the questions I ask about what the system, tests, and evals are actually doing,
and what changed because of the answer. Not graded directly, but it is the source for:
- G8 "what was built, and why"
- C7 "a score you can defend" (feeds `evals/RATIONALE.md` once it exists)
- Part 3 "report what you found — including what failed"

Format: one entry per question. Keep answers short; link to the decision, eval, or test that
came out of it. A question that changed nothing is still worth recording.

---

## Template

### YYYY-MM-DD — <question in one line>
**Context:** what I was looking at (eval name, test file, trace id, notebook).
**Answer:**
**What changed:** D-0xx / new eval `...` / test `...` / nothing — and why.
**Follow-up question, if any:**

---

## Entries

### 2026-09-13 — Does the rubric require a record of my inputs and questions?
**Context:** planning; deciding whether to keep this log at all.
**Answer:** Not the questions themselves. It requires their products: golden-set cases as
"your own cases" (Part 3), failures reported (Part 3), a defensible score per eval (C7), the
"why" behind the build (G8), and a budget a human chose (G7).
**What changed:** created this file; `evals/RATIONALE.md` planned for when `evals/` exists.
**Follow-up:** for every passing eval ask "what would have to be true for this to pass and the
system still be wrong?" — the answer is usually a missing eval.
