# Choose a worker and effort

Use explicit user choices and available, authorized models. This guide is optional, not an allowlist. Keep the coordinator's model and effort unless asked to change them. Keep work local when delegation would duplicate the coordinator's reasoning.

## Cost and acceptance

Choose a model after identifying work that delegation removes from the coordinator. Compare the complete attempt, including preparation, worker context, checking, likely repair and integration, with local completion. Use comparable accepted task outcomes when available; do not invent a precise saving from a benchmark score or token price.

Match the worker to the uncertainty remaining: retrieval or a fixed transformation, bounded implementation, or reasoning about unfamiliar behavior. A worker can own a whole bounded change, including navigation, local design, shared helpers, tests and corrections. Supply required behavior, interfaces, constraints and acceptance checks; leave implementation choices with the owner.

Choose effort for the remaining uncertainty and the consequences of a missed defect. Raise it for a concrete unresolved need, not automatically for implementation or review. Equal effort labels across models do not establish equal reasoning or cost. Lower effort can reduce generated tokens; it does not itself lower per-token prices. API estimates are not subscription allowance rates.

## Current model routes

- **GPT-6.1 Sol:** use the explicit ID `gpt-6.1-sol` through a supported Codex route. Select an effort exposed by that environment and confirm effective settings when available. [OpenAI model documentation](https://developers.openai.com/api/docs/models/gpt-6.1-sol) establishes the model ID; account and runtime availability still matter.
- **Claude Opus 5.5:** use `claude-opus-5-5` through an authorized Claude route. Read [claude-cli.md](claude-cli.md) for the bundled wrapper. [Claude Code model configuration](https://code.claude.com/docs/en/model-config) documents the explicit ID and current effort controls. Verify installed CLI support and account access before dispatch.
- **Other models:** use a supported, authorized route with explicit settings. External CLIs need their own account and billing authorization. Do not pass provider-qualified external IDs to native Codex workers or the Claude wrapper. Use an existing delegation procedure for isolation, completion delivery and saved handoffs.

These routes do not establish a task-specific effort recommendation, cost ranking or reviewer qualification. Use current documentation for support and comparable accepted outcomes for suitability.

## Diagnosis and review

For diagnosis, assign a specific failure and require a reproduction or discriminating check. Derive expected behavior from the contract, not the implementation. Keep unsettled cross-system requirements and final acceptance with the coordinator.

Delegate substantive review only when its coverage or independence justifies the cost and there is evidence it detects the relevant error types. Assign review and correction verification together. Passing worker-written tests alone is insufficient; verify consequential interactions, including affected callers and reused helpers. Finding a defect is successful review, not an automatic reason to change models.

Avoid a fresh external session for a tiny change. Retain a useful owner for bounded corrections when its approach remains sound. Reassess substantive failures rather than following an automatic model ladder.
