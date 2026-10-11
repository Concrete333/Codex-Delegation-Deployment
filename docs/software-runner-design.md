# Software runner: design and qualification

## Purpose

Move predictable lifecycle work out of paid model turns: dispatch one configured worker, wait for process completion, run already-chosen checks and save a compact receipt. The coordinator still chooses the assignment/model and owns substantive verification, correction decisions and final acceptance.

This is an experimental optional adapter, not a replacement for an efficient native completion wait. The runner has no model-based scheduler, automatic retries, reviewer loop, issue tracker, daemon, dashboard or team topology. It starts one fresh worker per request. One active claim per shared state directory limits concurrency and prevents accidental redispatch; there is no machine-wide ownership service.

## What we took from Symphony

Source inspected at commit [`e0ccc83720a42a600a53b61c5f8d3e518bebe1db`](https://github.com/openai/symphony/tree/e0ccc83720a42a600a53b61c5f8d3e518bebe1db). This implementation is original; it does not vendor Symphony code or launch Symphony.

| Source | Applied lesson |
| --- | --- |
| [Orchestrator](https://github.com/openai/symphony/blob/e0ccc83720a42a600a53b61c5f8d3e518bebe1db/elixir/lib/symphony_elixir/orchestrator.ex) | Software tracks claims and completion. Model turns need not own routine scheduling. Our single-slot claim deliberately omits its issue polling and retry machinery. |
| [Agent runner](https://github.com/openai/symphony/blob/e0ccc83720a42a600a53b61c5f8d3e518bebe1db/elixir/lib/symphony_elixir/agent_runner.ex) | Separate worker execution from orchestration state and preserve results. We omit automatic continuation turns and tracker-driven follow-up. |
| [Specification](https://github.com/openai/symphony/blob/e0ccc83720a42a600a53b61c5f8d3e518bebe1db/SPEC.md) | Make lifecycle, workspace and trust boundaries explicit. Process completion is distinct from the business outcome. |

The adapters use [Codex non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode) and [Claude programmatic mode](https://code.claude.com/docs/en/headless). Codex returns JSONL events and a schema-constrained last message; Claude returns JSON with `structured_output`. The existing Claude wrapper supplies its launch, authentication, environment and cleanup helpers.

## Acceptance and accounting boundaries

The software can establish that a process returned, a structured handoff was received, declared files had particular hashes and specific commands exited with particular codes. It cannot establish that the requirement was interpreted correctly, the tests were sufficient or the worker's risk assessment was sound. `ready_for_review` always carries `accepted: false`.

Prescribed checker/contract files can be fingerprinted before work and compared before/after checks. Artifact fingerprints tie observed checks to listed file contents. Neither feature protects undeclared files or replaces ownership and substantive verification. The runner is not itself a security sandbox. Codex checks now run under the worker's sandbox profile; Claude checks execute directly. Claude profiles and Codex sandboxes are separate provider controls.

Timeouts and interruptions preserve logs and retain ownership for explicit recovery even when tree termination reports success. A hard process crash leaves the claim intact. No stale PID lock is automatically stolen. Recorded-PID checks cannot prove all detached descendants have stopped; recovery requires inspection. Multiple state directories or unrelated processes can bypass the cooperative claim.

Prompts go through stdin. Full evidence is saved once; terminal output contains paths and small factual summaries. Requested settings and reported telemetry remain distinct. Codex usage does not consistently expose effective effort or a dollar cost, so these are not invented. Claude costs are client-side API-equivalent estimates, not a subscription allowance meter. Account-side overflow and worker/command activity outside the declared contract are not bounded by an allowance cap here.

## Offline verification

The offline suite launches deterministic Python fixtures through the real subprocess/log/check lifecycle. It covers success, partial/blocker reports, malformed output, CLI failure, missing/multiple completion events, permission denials, cost-limit returns, timeouts, preserved partial logs, claim retention, duplicate requests, changed request IDs, protected-checker changes, artifact changes during checks, compact output with full evidence retention, authentication guards, explicit configuration forwarding and invalid input. Existing Claude wrapper regression tests run alongside it.

```powershell
python -B -X utf8 -m unittest discover -s tests -q
```

These checks make no model calls and do not establish account/model availability or end-to-end paid-worker behavior. Help parsing does not validate every runtime configuration value. Host wakeups, real provider output, native sandbox behavior and cost through acceptance require a controlled live test.

## Next comparison

Use the same frozen task, starting files, worker model/effort, assignment and handoff contract, substantive reviewer and acceptance checks in both arms. Record actual prompts and runtime-added context rather than assuming the two launch paths are context-equivalent:

1. Native delegation and completion wait.
2. This runner owns launch, completion bookkeeping and the same final check commands.

Count all coordinator/worker tokens, cache categories, check/review work, repairs and failure costs through acceptance. Score consequential misses and false alarms against held-out cases; do not let fewer checks manufacture savings. Repeat the pair before claiming an advantage. Do not change routing, shorten policy, substitute a reviewer or add retries in the same comparison.

The tool itself does not yet establish savings. Native event waits already avoid one polling failure mode, and a separate CLI worker can add context/startup overhead. Its hypothesis is narrower: software can remove predictable coordinator work without lowering the acceptance standard.
