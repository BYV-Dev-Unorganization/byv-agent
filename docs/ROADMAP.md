# byv-agent Development Roadmap

**Important:** This roadmap intentionally contains no day, week, month, sprint-length, or calendar-duration estimates. Progress is determined by deliverables and acceptance gates, not elapsed time.

## Roadmap operating rules

- Complete phases in dependency order unless a documented dependency permits parallel work.
- A phase is not complete because code exists; its acceptance criteria and tests must pass.
- A feature may be marked implemented only when behavior, documentation, diagnostics, and failure paths are covered.
- Track status in the project issue board or a dedicated implementation tracker.
- Do not mark a compatibility feature “supported” until its fixture and runtime conformance tests pass.
- Security gates apply throughout development, not only at the final release.

## Phase 0 — Requirements and repository foundation

Deliverables:
- Product scope and terminology are approved.
- Repository, package conventions, contribution guide, license decision, and CI baseline are established.
- Architecture decision records are maintained.
- Strict TypeScript configuration, Bun package/workspace setup, linting, formatting, bun test framework, and build pipeline exist.
- Pin Bun version and document supported local development platforms.
- Add a minimal standalone compile smoke test.
- Create build and distribution scripts for the first Bun-supported target.
- Dependency policy and version pinning are documented.
- Security reporting and secret-handling rules are documented.

Exit gate:
- Clean checkout can install dependencies, validate formatting/types, run tests, and build all workspace packages.

## Phase 1 — Core contracts and CLI shell

Deliverables:
- Canonical message/content model.
- Canonical provider event model.
- Canonical tool schema and tool registry.
- Session controller interface.
- Agent state machine and typed errors.
- CLI command parser and interactive prompt shell.
- Structured logging and trace identifiers.
- Configuration schema and migration mechanism.

Exit gate:
- Fake provider can stream a deterministic response through the CLI.
- State transitions, cancellation, and error handling are unit-tested.

## Phase 2 — Provider and model system

Deliverables:
- OpenAI-compatible Chat Completions adapter.
- Anthropic Messages adapter.
- Provider custom base URL, key references, headers, timeout, and retry configuration.
- Model catalog and aliases.
- Capability registry.
- Streaming and tool-call normalization.
- Usage normalization.
- Effort/reasoning capability mapping.
- Provider health check and safe fallback.
- Optional native OpenAI Responses adapter as a separate protocol.

Exit gate:
- Provider contract suite passes for text, stream, tool calls, tool results, usage, effort mapping, errors, and cancellation.
- Unsupported provider capabilities are accurately reported.

## Phase 3 — Core agent loop

Deliverables:
- Model request builder.
- Tool-call parser and schema validation.
- Permission gate integration.
- Tool execution round trip.
- Sequential and supported parallel tool calls.
- Retry and recovery policies.
- Planning and normal modes.
- Stop, interrupt, continuation, and bounded autonomous loop.
- Task budget and stop conditions.
- Final response and task evidence structure.

Exit gate:
- Agent completes deterministic multi-step fake-provider tasks using fake tools.
- Invalid, denied, timed-out, and cancelled tool calls never execute unsafely.

## Phase 4 — Local coding runtime

Deliverables:
- Workspace discovery and boundary enforcement.
- Filesystem read/write/edit/search/list tools.
- Terminal runner with persistent and one-shot commands.
- Process management, output streaming, cancellation, and timeout.
- Git read and mutation tools with permission controls.
- Test and build commands.
- File change journal and initial checkpoint support.

Exit gate:
- Agent can inspect a repository, edit files, run tests, report diffs, and preserve unrelated user changes.
- Cross-platform runner tests cover supported operating systems/shells.

## Phase 5 — Context, sessions, memory, and compaction

Deliverables:
- CLAUDE.md and AGENTS.md loading.
- Project instructions and rules resolver.
- Ignore-aware indexing and search.
- Context budgets and token estimation.
- Context usage monitor.
- Session persistence and resume.
- Append-only transcript and tool artifact storage.
- Manual compaction.
- Automatic compaction with threshold/target configuration.
- Compaction transaction and failure recovery.
- PreCompact/PostCompact lifecycle integration.
- Project/user/session memory with inspect/edit/delete controls.

Exit gate:
- Long-session fixture triggers auto-compaction before context exhaustion.
- Compaction preserves active goal, instructions, decisions, tool pairing, modified files, tests, blockers, and pending actions.
- Original transcript remains recoverable.
- Restart and resume restore the correct active context.

## Phase 6 — Effort, usage, and routing

Deliverables:
- Effort levels and precedence.
- Provider/model-specific mapping.
- Per-agent and per-task overrides.
- Context/model-aware routing.
- Token and cost tracking.
- Pricing catalog with source/effective date.
- Prompt cache accounting where returned.
- Usage commands and budget warnings/stops.

Exit gate:
- Effective effort is visible and accurately mapped or reported unsupported.
- Usage reports distinguish provider-reported values from estimates.
- Budget exhaustion stops work safely.

## Phase 7 — Claude Code Skills and instruction compatibility

Deliverables:
- Skill discovery by scope.
- SKILL.md YAML parser and validation.
- Frontmatter support and unknown-field diagnostics.
- Lazy description/body loading.
- Manual and model-triggered invocation rules.
- Argument substitution and autocomplete hints.
- Supporting files.
- Namespaces and conflict resolution.
- Command file compatibility.
- Reload and diagnostics.

Exit gate:
- Compatibility fixtures pass for valid/invalid fields, invocation controls, arguments, resource paths, scope collisions, and plugin namespace.

## Phase 8 — MCP compatibility

Deliverables:
- .mcp.json parser and scope resolution.
- Stdio transport.
- Streamable HTTP transport.
- Legacy SSE compatibility.
- Optional WebSocket support if included in the selected compatibility baseline.
- Tool/resource/prompt discovery.
- Authentication and secure headers.
- Dynamic server/tool updates.
- Reconnection and server lifecycle management.
- Per-agent MCP scope and permission integration.

Exit gate:
- Local and remote fixture servers pass tool, resource, prompt, error, reconnect, timeout, and shutdown tests.
- Credentials are redacted and untrusted MCP output remains behind policy.

## Phase 9 — Plugins, marketplace, and hooks

Deliverables:
- Plugin manifest loader and validator.
- Plugin component discovery.
- Marketplace catalog parser.
- Local/Git sources.
- Install/update/enable/disable/uninstall/rollback.
- Namespaced Skills and Agents.
- MCP and hook loading.
- Hook event registry and dispatcher.
- Command/HTTP/prompt/agent/MCP-tool handler support by conformance level.
- Trust review, permissions, audit, and diagnostics.

Exit gate:
- Example plugin and fixture marketplace install safely.
- No plugin code executes merely because it is discovered.
- Hook match, payload, output, timeout, and decision semantics pass fixtures.

## Phase 10 — Subagents and multi-agent execution

Deliverables:
- Claude-compatible agent frontmatter parser.
- Built-in agent roles.
- Per-agent provider/model/effort/tool/permission/context settings.
- Task delegation and structured result contract.
- Parallel execution and concurrency limits.
- Cancellation and partial-result handling.
- Subagent memory.
- Worktree isolation for concurrent writers.
- Parent integration and verification.

Exit gate:
- Parallel read-only agents can produce correlated results.
- Parallel writers cannot silently overwrite one another.
- Parent verifies integrated changes and test status.

## Phase 11 — Advanced coding workflows

Deliverables:
- Plan approval workflow.
- Debug workflow and root-cause report.
- Read-only code review workflow.
- Autonomous goal/checklist engine.
- LSP client.
- Tree-sitter parsing.
- Symbol/reference/diagnostic tools.
- Git worktree and checkpoint/rewind UI.
- Regression verification.
- Diff viewer.

Exit gate:
- Coding evaluation suite demonstrates task completion with evidence and no unapproved destructive operations.

## Phase 12 — Background, queue, and browser tools

Deliverables:
- Background shell tasks.
- Background subagents.
- Persistent job queue.
- Message queue and mid-turn steering.
- Task pause/resume/cancel.
- Stall detection and notifications.
- Browser search/read/Playwright tools.
- Screenshot and frontend verification.
- Browser context isolation and network policy.

Exit gate:
- Background job state survives restart.
- Queued prompts are ordered and can be cancelled.
- Browser tasks respect user authorization and network restrictions.

## Phase 13 — Scheduler and integration API

Deliverables:
- One-time and recurring scheduled tasks.
- Timezone-aware schedule parsing.
- Event-triggered jobs and webhooks.
- Job deduplication, idempotency, concurrency policy, and missed-run handling.
- Approval policy for unattended side-effecting tasks.
- Job history, logs, notifications, and budgets.
- Local authenticated API and event stream for approved session/task control.
- API versioning and access scopes.

Exit gate:
- Scheduled jobs respect permissions, budget, sandbox, and audit systems.
- Duplicate or overlapping jobs follow explicit idempotency and concurrency rules.
- API operations are authenticated, scoped, and auditable.

## Phase 14 — Sandbox and remote execution

Deliverables:
- Container runner.
- Remote runner contract.
- Worker registration/authentication.
- Workspace snapshot/artifact transfer.
- Short-lived credentials.
- Network/filesystem/resource policy.
- Remote task logs and heartbeat.
- Cancellation and recovery.
- Runner diagnostics.

Exit gate:
- A task can run in an isolated runner with scoped workspace and credentials.
- Remote failures do not corrupt local session or transcript state.

## Phase 15 — Release hardening and ecosystem

Deliverables:
- Install and upgrade packaging.
- Supported OS matrix.
- Full documentation and examples.
- Provider and extension compatibility matrix.
- Reproducible CI.
- Security review and threat-model test suite.
- Model/provider evaluation harness.
- Performance and resource tests.
- Migration and backup/restore.
- Release notes and versioning policy.

Exit gate:
- All release acceptance criteria pass.
- Unsupported compatibility features are documented.
- No known critical security, data-loss, or secret-disclosure defects remain.

## Parallel work guidance

The following work may proceed in parallel after contracts are stable:
- Provider adapters and CLI display.
- Tool adapters and context indexing.
- Skills parser and MCP configuration parser.
- Plugin manifest parser and hook event registry.
- Storage migrations and test harness.
- Security review and compatibility fixture authoring.

Do not parallelize implementation of shared canonical types without ownership and review. Changes to provider event, tool, session, or extension contracts require cross-package review.

## Roadmap status template

For each deliverable, record:
- Status: not started / in progress / blocked / implemented / verified.
- Owner.
- Dependencies.
- Relevant issue/PR.
- Test/evaluation evidence.
- Known limitations.
- Compatibility baseline if relevant.

No calendar estimate is required or permitted in this roadmap.
