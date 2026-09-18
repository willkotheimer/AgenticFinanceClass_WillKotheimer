# Links

Resources shared in the course (Day 4, secure agent architectures) and elsewhere.

## Security / red-teaming

- https://github.com/confident-ai/deepteam — LLM red-teaming framework (prompt injection, jailbreak, PII leakage test suites). Candidate for the security evals.

## Sandboxing / isolated execution

- https://github.com/pydantic/monty — Pydantic's sandboxed Python runtime for programmatic tool calling (Day 4 "Programmatic Tool Calling (Monty)" notebook).
- https://e2b.dev/ — hosted sandboxes for agent code execution (Day 4 "LangGraph + E2B Sandbox" notebook).
- https://firecracker-microvm.github.io/ — Firecracker microVMs; the isolation layer behind services like E2B.
- https://github.com/google/gvisor — gVisor application kernel for container sandboxing; a lighter alternative to microVMs.

## Course material

- `..\packt_agent_engineering_finance\` — course notebooks; `day_one\SKILL_consulting.md` and `SKILL_frontend.md` are the SKILL.md examples for component C6.

## Instructor repos (cloned to `..\`, 2026-09-18)

- `packt_agent_engineering_finance` — the course (private; bundle in `sourceackups\`).
- `harness_engineering` — code for her O'Reilly book *Harness Engineering*; G1 vocabulary.
- `ai-agents-the-definitive-guide` — her AI agents book; Day 3/4 patterns.
- `SkillOpt` — optimiser for `SKILL.md` files; C6.
- `AgensFlow` — her multi-agent coordination framework; Day 4 Block 2 topologies.
- `ORM-self-improving-ai-agents-course` — her other course; eval / RL overlap.
