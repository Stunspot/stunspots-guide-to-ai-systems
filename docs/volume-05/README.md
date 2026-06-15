# Volume 5 — Agentic Systems and Tool-Using Architectures

*How static generators become actors, and how to keep them from becoming raccoons with API keys.*

## Reports

- [AI-ENG-M — Agentic Orchestration: Autonomy Boundaries, Loop Budgets & Termination](./ai-eng-m-agentic-orchestration.md)
    Covers agent loops, planning, reflection, decomposition, memory use, tool selection, scratchpads, state machines, graph-based workflows, loop budgets, tool budgets, timeout policies, escalation triggers, and termination conditions. Treats autonomy as a bounded architectural affordance, not a vibe.

- [AI-ENG-N — Tool Contracts: Function Calling, Schemas, Validation & Idempotency](./ai-eng-n-tool-contracts.md)
    Covers tool interfaces, schema design, argument validation, retry logic, repair loops, deterministic wrappers, idempotency, side-effect control, transactional boundaries, confirmation gates, and safe execution patterns. Establishes the principle that probabilistic actors need deterministic edges.

- [AI-ENG-O — Action Verification: Planning, Execution, Observation & Recovery](./ai-eng-o-action-verification.md)
    Covers verifying that tools did what the model thinks they did. Includes post-action checks, state reconciliation, rollback paths, partial failure handling, compensating actions, tool-result grounding, and preventing hallucinated success.

[← Back to Canon Map](../canon-map.md)