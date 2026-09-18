# Capstone requirements

The capstone brief as given in the course. This is the spec we are graded against;
everything else in this folder serves it.

## The brief

> Submit a finance agent workflow that demonstrates harness engineering, skill design,
> retrieval, tool use, evaluation, and security controls.

- Suggested domains: document analysis, earnings-call analysis, portfolio review, compliance
  flagging, financial chart interpretation, or any other finance-relevant workflow.
- Must include **at least three** of the eight components below.
- We pick the workflow and the domain.

## The eight components

| # | Component | Course block | What it means |
|---|---|---|---|
| C1 | Document retrieval and grounding | Days 1–2 | Multimodal retrieval over filings and reports, with figures attributable to a source |
| C2 | Table or chart interpretation | Day 2 | Values read out of rendered tables and charts into structured form |
| C3 | Earnings call or audio transcript analysis | Day 2 | Audio or transcript input treated as a first-class modality |
| C4 | Time series or market data analysis | Day 2 | Forecasting or market-data models used as tools inside the workflow |
| C5 | Tool-using agent workflow | Days 1 & 4 | Governed tool calls with declared arguments, boundaries and failure handling |
| C6 | SKILL.md-based skill definition | Day 1 | At least one reusable skill defined as a structured skill file |
| C7 | Evaluation and tracing | Day 3 | Spans across the run, deterministic checks, and a score you can defend |
| C8 | Security controls | Day 4 | Sandboxing, tool restrictions, approval gates, or prompt-injection mitigation |

## Timeline and deliverables

Six weeks total, one checkpoint in the middle.

### Part 1 — Map the workflow (weeks 1–2) — **reviewed before we build**
- Identify tasks, roles, decision points, agent responsibilities, context requirements, tool
  dependencies, memory needs, and human checkpoints.
- Output: **one to two pages**: a short introduction to what the project is and what workflow
  it solves, a **diagram** of the workflow, and the workflow mapped into agentic components
  with the components (C1–C8) we intend to build.
- **No code yet.**

### Checkpoint — review and go-ahead
- Feedback and confirmation. The four build weeks start *after* feedback arrives.

### Part 2 — Design the harness
- For each chosen component: architecture, assumptions, risks, evaluation criteria, failure
  modes, cost considerations, security boundaries, monitoring needs, governance requirements.
- Output: a design document naming **what the system may and may not do**.

### Part 3 — Build and evaluate
- Implement, instrument, run against our own cases, report what we found — **including what
  failed**.
- Output: a working notebook or repository, a trace, and an evaluation result.
- Earlier submissions welcome.

### Submission
- Private repo: add the instructor as a collaborator. Public repo: send the link.

## Grading — eight criteria

| # | Criterion | The question being asked |
|---|---|---|
| G1 | Agent architecture and harness design | Is it a harness around the model, not a chain of calls? |
| G2 | Correct use of multimodal inputs | Do tables, charts, time series, audio each get handling that suits the input — not one generic path? |
| G3 | Skill and context design | Are skills reusable and context boundaries deliberate? |
| G4 | Reliability of tool use and retrieval | Are tool calls and retrieval bounded, classified on failure, and recoverable? |
| G5 | Evaluation and observability | Can the run be traced, scored, and replayed? |
| G6 | Security and governance | Are boundaries, approvals, and audit trails present and enforced? |
| G7 | Loop design and control | Is the run bounded: declared termination, retry and escalation paths, a budget somebody chose? |
| G8 | Clarity of documentation | Could someone else read the submission and understand what was built, and why? |
