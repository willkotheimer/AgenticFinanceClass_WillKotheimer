# Instructor concepts

Concepts taken from the instructor's public repos (`harness_engineering`,
`ai-agents-the-definitive-guide`). Concepts only. We write our own code; her code may be
adapted with attribution where it fits (D-029).

## What a harness is

- Six components around the model. Each has one job:
  - Observability → records what happened.
  - Governance → constrains which transitions are allowed.
  - Execution → turns a proposal into an effect.
  - Verification → reads evidence and state; accepts, corrects, or rejects.
  - Adaptation → reuses only accepted state.
  - Orchestration → moves execution through state.
- The model proposes. The harness decides.
- Take the shape: named components with seams between them. Rebuild the insides.

## State is the connective tissue

- Four kinds of state:
  - Model context → what this model call sees.
  - Workflow state → where the run is: step, retry count, pending approval, next transition.
  - Operational state → the world the harness reads or changes.
  - Durable state → what survives the process: checkpoints, approvals, verified results.
- The harness decides which state is exposed, which can change, and which persists.
- Workflow state must be harness-owned and durable. A run id is a locator, not a credential.

## Boundaries a run must cross

- Process → checkpoint and rehydrate.
- Time → a human approval arrives later; the run resumes.
- Effect → commit exactly once.
- Agent → no shared model memory; artifacts cross, context does not.

## Evidence over assertion

- Ablation: remove one component, replay the same recorded trajectory, observe what fails.
- Recorded trajectories are the offline eval loop. They test the harness, not the model.
- Reliability lives in the interactions, not in the boxes.

## Contracts

- Pydantic models at every boundary. Fail fast. Do not repair invalid data.
- Instructor = recovery at the source (retry with the validation error).
- Pin a run to the policy version it started with. Store the policy hash in the record.
- Storage schema and tool-output schema evolve independently.

## Evaluation

- Scenarios with defined success, edge cases, and explicit stress dimensions.
- Deterministic checks first; LLM judge second; both scored.
- Sort failures for expert review. Export what a human must check.

## How this shapes our design

- Four kinds of state → four places in our graph: message context, LangGraph state,
  EDGAR + cache, checkpointer.
- Six components → our modules: `trace`, `policy`, `tools`, `review`, `cache`, `graph`.
- Ablation → our single-agent baseline and the "remove the reviewer" test.
- Recorded trajectories → our pytest fixtures.
