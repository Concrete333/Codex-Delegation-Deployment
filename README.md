# Agent Deployment for Codex

Reduce unnecessary Codex usage while waiting for commands or coordinating other agents. **You do not need subagents to use it.**

If Codex starts a slow test suite, build or database script, it can keep calling tools to ask whether the job has finished. Each model turn adds overhead. The skill guides Codex to use a completion notification, let the command run, and check its saved result when it finishes.

For larger tasks, it helps Codex hand over bounded work and review completed results without duplicating the worker's investigation. Delegation is optional: checking and correcting a worker can cost more than doing the task yourself.

[Install](#install) · [Use](#use) · [Scripts and test suites](#scripts-and-test-suites) · [Agent delegation](#agent-delegation) · [Choosing models](#choosing-models) · [Verification](#verification)

## What it changes

| When Codex is… | The skill guides it to… |
| --- | --- |
| Waiting for tests, a build, a script or a database job | Use supported completion notifications instead of repeatedly checking unchanged logs. |
| Considering another AI agent | Identify work it can actually hand over and how to verify the result. Keep small or tightly coupled work with one owner. |
| Waiting for a worker | Leave unfinished edits alone, do independent useful work, and review completed handoffs or agreed checkpoints. |
| Receiving a result | Check saved evidence before continuing. A notification or a worker saying “done” does not establish success. |

Commands that fit one supported foreground tool wait stay direct. Longer work uses a supported completion wait; safely detached work needing a later wake-up can use the bundled notification helper. If neither route is available, Codex explains what is running and asks you to check back.

## Install

Use Git to clone the skill. The bundled completion helper needs Python 3.10+ and a native Codex CLI supporting `codex queue`. Subagents are optional; choosing their models and reasoning efforts requires a Codex environment that exposes those controls. The skill does not unlock models or change account access.

For a new user-level installation, use the skills location in [OpenAI's documentation](https://learn.chatgpt.com/docs/build-skills).

macOS or Linux:

```bash
git clone https://github.com/Concrete333/Codex-Delegation-Deployment.git ~/.agents/skills/agent-deployment
```

Windows PowerShell:

```powershell
git clone https://github.com/Concrete333/Codex-Delegation-Deployment.git "$env:USERPROFILE\.agents\skills\agent-deployment"
```

If Codex already discovers an installation elsewhere, update that copy instead of creating a duplicate. Restart Codex if the skill does not appear.

## Use

For a long-running command:

```text
$agent-deployment Run the full test suite. Use completion notification
instead of repeated status checks, preserve safety timeouts, and verify
the results before continuing. No subagents are needed.
```

For a task where delegation may help:

```text
$agent-deployment Use subagents where helpful to fix duplicate records
in offline sync. Preserve the API and add regression tests.
```

To allow Claude workers:

```text
$agent-deployment You may use my installed Claude CLI and subscription
for this task. Choose appropriate models and efforts, and verify the result.
Do not enable paid overflow.
```

Codex can select the skill when a request matches its description. Explicitly naming it is the clearest way to request it. Skill selection does not itself authorize delegation, edits or additional spending.

### Optional reminder in AGENTS.md

To make the pre-launch reminder available in ordinary coding tasks, add this paragraph to your project or global `AGENTS.md` after installing the skill:

```text
Before launching an expected long-running job, including a test suite, use the
`agent-deployment` skill's waiting guide to establish native completion or preflight
and arm its queue helper. Known-short commands stay direct, even full suites; use
existing timing evidence, not a separate timing run. This applies without subagents.
Do not launch first and default to manual check-in later; if notification is
unavailable, state the specific limitation.
```

The skill contains the full waiting and recovery rules; this reminder helps Codex select them before launch. Neither file guarantees instruction compliance or notification delivery.

## Scripts and test suites

Codex can launch a command that safely continues in the background, end its turn, and receive one queued message after the helper saves the terminal result. This works for tests, builds, data-processing jobs and external worker CLIs where the host supports it. Each detached command produces a visible follow-up, which can arrive while Codex is still working; use foreground waiting when it already collects the result. Delayed messages for already-handled receipts should be ignored silently.

The [waiting guide](references/waiting.md) covers setup and recovery. Commands retain safety timeouts and cleanup controls. Codex verifies saved results before continuing; a notification alone does not establish success. Already-running jobs are not restarted just to add notifications. Script-only tasks skip the delegation and model-selection guides.

### Optional native-agent wait configuration

To prevent short timeout loops when waiting for native subagents, add or merge this into your Codex `config.toml`:

```toml
[features.multi_agent_v2]
enabled = true
min_wait_timeout_ms = 1500000
default_wait_timeout_ms = 1500000
max_wait_timeout_ms = 1500000
```

The timeout is 25 minutes; supported completion events can end the wait sooner. Confirm that your installed runtime supports these settings and respect your host's wait limits.

Use native waiting for subagents when it works. Use the queue helper for suitable background commands or external workers without an equivalent completion route.

## Agent delegation

Your current Codex agent is the coordinator: it defines the outcome, owns consequential decisions and checks the final result. A worker is another AI agent given a bounded assignment, such as implementing a component with settled interfaces or searching specified source files.

Before dispatch, Codex considers preparation, worker execution, review and likely corrections. The worker should remove work from the coordinator, owning implementation and tests within settled boundaries. The coordinator leaves partial edits alone and batches corrections after completed handoffs or agreed checkpoints.

Keep a coupled change with one implementation owner, which can be the worker. The coordinator settles consequential requirements and checks the result, including affected callers and reused helpers. Passing worker-written tests alone is insufficient. Bounded defects go back to the worker; failed approaches and newly exposed design decisions return to the coordinator. Small work often costs less to finish locally.

The core guidance has no model allowlist. Your current agent keeps its model and effort unless you ask to change them, and worker choices respect your available tools and preferences. Required verification stays in place even when a cheaper worker is used.


## Choosing models

The optional [model-selection guide](references/model-selection.md) covers model choice, effort and total cost through acceptance. Codex skips it when you have already chosen the worker model and effort. Use available, authorized models and verify effective settings; the guide is not an allowlist.

Current examples use GPT-6.1 Sol (`gpt-6.1-sol`) and Claude Opus 5.5 (`claude-opus-5-5`). The guide links their official documentation. Choose effort for the remaining uncertainty, acceptance checks and consequences of a missed defect. Compare the complete accepted result, including preparation and corrections, against local completion. Keep the current coordinator's model and effort unless you ask to change them.

For external workers, the separate [Kilo Delegator skill](https://github.com/Concrete333/Codex-Kilo-Delegation) supplies isolated worktrees, completion notifications and saved handoffs. It needs its own authenticated account and authorized billing. This skill does not require Kilo.

## Optional software runner

The included [deployment runner](references/software-runner.md) handles one configured CLI worker, process completion, predefined checks and saved execution records. It forwards an explicit model and effort to Codex or the Claude wrapper; the selected CLI and account must support them. Python owns the bookkeeping; your coordinator owns review and acceptance.

The runner prevents accidental repeats of the same request, keeps full logs out of routine model context, detects changes to declared checker files and retains ownership after interrupted work. It does not choose models, retry automatically or treat passing tests as acceptance.

This route is experimental and optional. Use native subagents when their completion wait is sufficient. The [runner guide](references/software-runner.md#sandboxed-coordinator-without-cli-authentication) also covers coordinators whose sandbox cannot access CLI authentication. See the [design notes](docs/software-runner-design.md) for scope and qualification, including the use of OpenAI's Symphony as a design reference.

## Claude CLI workers

Codex calls your installed Claude Code CLI through the included Python wrapper; no MCP server required. The wrapper accepts an explicit Claude model and effort, with read-only or editing profiles, session resumption and bounded batches of independent workers.

Requirements: Python 3.10+, an installed Claude Code CLI supporting the requested flags, and a logged-in Claude account with access to the selected model. Verify model and effort support before dispatch.

See [the Claude CLI guide](references/claude-cli.md) for setup, context controls, request format and session continuation. Compact results link to complete local execution records.

Important limits:

- Tool restrictions are not an operating-system sandbox. The target repository and inherited Claude context must be trusted.
- Usage records contain estimates, not remaining subscription allowance. The wrapper cannot disable account-side paid overflow.
- A successful CLI exit is not accepted work. The coordinator still checks results, artifacts and unresolved risks.
- Concurrent editing jobs need separate checkouts and independent shared state. A worker timeout terminates work; a coordinator wait timeout does not.

## What agents load

| File | Purpose | Loaded during ordinary deployment? |
| --- | --- | --- |
| [SKILL.md](SKILL.md) | Routes command-only tasks to waiting guidance; helps decide whether to delegate other work | When the skill is selected |
| [Waiting guide](references/waiting.md) | Completion notifications, saved results and recovery for commands and workers | For long-running work needing a completion route |
| [Delegation guide](references/delegation.md) | Ownership, verification, continuation and waiting | Only when delegation is useful |
| [Model-selection guide](references/model-selection.md) | Model routes, total cost and effort choices | When delegation is useful and the worker model or effort still needs choosing |
| [Claude CLI guide](references/claude-cli.md) | How to invoke and manage Claude workers | For Claude delegation only |
| [Coordination sources](docs/agent-coordination-references.md) and [runner design](docs/software-runner-design.md) | Audits and skill improvement | No |

Workers receive their assignment and relevant project evidence, not the entire routing guide. Audits and maintenance may read the relevant material in `docs/`.

For a Claude Code coordinator instead of Codex, see [Claude Agent Deployment](https://github.com/Concrete333/Claude-Agent-Deployment).

## Verification

The bundled offline tests exercise worker requests, permissions, process completion, receipts, checker integrity and host ownership. Run them from the repository root:

```text
python -m unittest discover -s tests
```

These checks cover tool behavior, not model quality, reviewer reliability or cost savings. Compare accepted outcomes on your own tasks before relying on a model or workflow advantage. Include preparation, review, corrections and failed attempts; API estimates are not subscription bills or quota savings.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) to improve or audit the skill. Runtime instructions live in `SKILL.md` and `references/`; contributor guidance and research stay outside ordinary skill use. The repository's `AGENTS.md` is for working on this repository, not a file to copy into your project. Use the [optional reminder](#optional-reminder-in-agentsmd) above instead.

## Credits

The [coordination reference list](docs/agent-coordination-references.md) records the research and engineering sources behind the model-independent ownership, verification, context and waiting principles.
