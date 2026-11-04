# byv-agent Acceptance Criteria

This document defines observable completion conditions. A feature is not complete until its behavior, edge cases, diagnostics, tests, and documentation meet the applicable criteria.

## 1. Provider and model acceptance

- Custom OpenAI-compatible base URL can be configured without source changes.
- Native Anthropic Messages provider can be configured.
- Secrets are referenced securely and never printed.
- Text and streaming responses work.
- Structured tool calls are parsed without flattening.
- Multiple and sequential tool calls round-trip correctly.
- Provider errors are classified.
- Timeout, cancellation, malformed stream, and rate-limit cases are tested.
- Model capability values are supported/unsupported/unknown.
- Unsupported effort, vision, tool, or cache capabilities are reported honestly.
- Model switching preserves valid session history or explains incompatible state.
- Fallback does not duplicate non-idempotent tool actions.

## 2. Effort and reasoning acceptance

- All canonical levels are recognized by configuration and CLI.
- Per-turn, task, agent, model, and global precedence is tested.
- Adapter mapping is verified for each provider/model profile.
- Effective effort and mapping state are visible.
- Unsupported native effort is not misrepresented.
- Provider-specific thinking controls remain separate from canonical effort.
- Subagent effort is independent and budgeted.

## 3. Agent loop acceptance

- Text-only completion works.
- Single tool call works.
- Multiple tool calls work.
- Tool result continuation works.
- Invalid tool arguments never execute.
- Denied tool calls never execute.
- Tool errors return controlled results.
- Cancellation reaches model stream and running process.
- Retry is bounded and respects idempotency.
- Completion is based on stop conditions and goal verification.
- Budget exhaustion produces a useful final status.

## 4. Context and compaction acceptance

- Context usage is visible.
- Model context limits and output reserve are accounted for.
- Provider-reported usage is distinguished from local estimation.
- Manual compact works.
- Auto-compact triggers at configured threshold.
- Per-model and per-agent thresholds work.
- Compaction does not break tool call/result pairing.
- Compaction preserves system/developer instructions, user constraints, active goal, plan, decisions, modified files, test results, failures, and pending actions.
- Full raw transcript and full tool artifacts remain available.
- Compaction transaction is atomic.
- Failed summarization/validation/storage leaves the previous active context intact.
- Agent resumes after successful compact without duplicate tool execution.
- PreCompact and PostCompact hooks have compatible input/output behavior.
- Context reset is never confused with compact.

## 5. Session and memory acceptance

- New session, list, resume, continue, clear, fork, branch, export, import, archive, and delete work according to documented semantics.
- Session state survives process restart.
- Raw transcript is recoverable.
- Memory can be inspected, edited, disabled, and deleted.
- Memory retrieval records provenance.
- Secrets and sensitive values are not saved by default.
- Subagent memory scopes are isolated.

## 6. Filesystem and terminal acceptance

- Files outside workspace cannot be accessed without permission.
- Path traversal and symlink escapes are tested.
- File writes are atomic where possible.
- Unrelated user changes are preserved.
- Large and binary files are handled safely.
- Terminal streams stdout and stderr.
- Exit code is captured.
- Timeouts and process-tree cancellation work.
- Background processes are identified and manageable.
- Environment variables are filtered.
- Destructive commands require approval.

## 7. Git and checkpoint acceptance

- Git status/diff/log are correct.
- Commit and branch operations are permission-controlled.
- Worktree isolation is tested.
- Conflicts are surfaced.
- Checkpoint can restore agent-owned changes.
- Rewind does not silently remove unrelated user edits.
- Reset, clean, force push, and destructive restore require explicit approval.

## 8. Skills compatibility acceptance

- Skill discovery works at supported scopes.
- YAML frontmatter at file start is parsed.
- Invalid frontmatter produces diagnostics.
- Known fields are applied correctly.
- Unknown fields are preserved/reported.
- Manual and automatic invocation controls work.
- Arguments and argument hints work.
- Supporting resources resolve within the skill root.
- Plugin namespace prevents collisions.
- Skill allowed-tools cannot bypass global policy.
- Lazy loading and reload invalidate cache correctly.

## 9. MCP compatibility acceptance

- .mcp.json parses and validates.
- Stdio server starts and stops.
- Streamable HTTP server connects.
- Legacy SSE compatibility is tested if enabled.
- WebSocket configuration is tested if enabled.
- Tools/resources/prompts are discovered and invoked.
- Dynamic tool changes are handled.
- Authentication and token refresh work where supported.
- Invalid or untrusted server is reported.
- Server timeout, disconnect, reconnect, and process cleanup work.
- MCP output cannot bypass permission or prompt-injection defenses.

## 10. Plugin and marketplace acceptance

- Plugin manifest validates.
- Plugin components are discovered from documented locations.
- Marketplace catalog parses.
- Local and Git sources are handled.
- Install/update/disable/remove/rollback work.
- Plugin scripts do not run on discovery.
- Plugin trust and permissions are visible.
- Unsafe path and dependency inputs are rejected.
- Unsupported plugin components generate explicit diagnostics.
- Example fixture plugin passes.

## 11. Hooks acceptance

- Hook configuration loads from supported scopes.
- Event registry is versioned.
- Matchers follow documented semantics.
- Command hook stdin/stdout/exit code contract is tested.
- HTTP, prompt, agent, and MCP-tool hooks are tested if claimed supported.
- Timeouts, cancellation, output limits, and event-specific failure behavior work.
- Hooks cannot bypass permission policy.
- PreCompact/PostCompact and tool lifecycle hooks are tested.

## 12. Subagent acceptance

- Agent Markdown metadata validates.
- Model, effort, tools, permissions, skills, MCP scope, and memory are applied.
- Agent task packet has defined scope and acceptance criteria.
- Parent receives structured result and artifacts.
- Cancellation and max-turn/maximum-budget behavior work.
- Parallel read tasks return correlated results.
- Parallel writes use isolation or locks.
- Parent verifies integration and tests.

## 13. Background and queue acceptance

- Background process can be inspected and stopped.
- Job status survives restart where persistence is promised.
- Queued user messages retain ordering.
- Immediate steering and interrupt are distinct from queue.
- Queue item can be removed.
- Background task failure is visible.
- `/btw` can be submitted during active generation, tool execution, and background waiting without entering the main queue.
- Side-question requests run independently from the main agent-loop lock and return without waiting for the main turn to finish.
- Side questions receive a frozen, redacted snapshot of committed context and no tool definitions.
- Tool calls emitted during a side question are rejected and never dispatched.
- Side prompts/answers are excluded from main transcript, active context, compaction, memory, task state, and queue by default.
- Side-request failures, cancellation, timeout, and provider rate limits do not affect the main task.
- Side-question tokens and cost are tracked separately from main-task usage.
- Dismissing an answer overlay preserves the main composer draft and does not cancel either the main turn or the side request.
- Concurrency limits, visible-history clearing, restart recovery, and explicit fork promotion are tested.
- Stall detection does not falsely mark long but active tasks as stalled.

## 14. Browser acceptance

- Browser contexts are isolated.
- User authentication is only through authorized sessions.
- Page content is marked untrusted.
- Network and download policies are enforced.
- Screenshot and DOM tools return bounded results.
- Browser task cancellation closes owned contexts.

## 15. Security acceptance

- Secret redaction tests pass.
- Permission-denied actions cannot execute.
- Project instructions cannot grant global access.
- Plugin/MCP trust prompts are enforced.
- Sandbox resource and network restrictions are tested.
- Audit log records all sensitive actions.
- Tool output size is limited.
- Remote runner receives scoped credentials only.
- No critical security or data-loss bug remains for release.

## 16. Reliability and observability acceptance

- Every operation has correlation identifiers.
- Logs redact secrets and private reasoning.
- Provider/tool latency and error metrics are available locally.
- Diagnostics can identify provider, model, MCP, Skills, plugin, and config failures.
- Telemetry is opt-in.
- Diagnostic bundle is redacted.
- Repeated task evaluation detects regressions.
- Documentation reflects actual support state.

## 17. Scheduler and integration API acceptance

- One-time and recurring jobs persist according to the configured policy.
- Schedule parsing is timezone-aware and handles daylight-saving changes where relevant.
- Missed-run behavior is explicit.
- Duplicate triggers are deduplicated when configured.
- Side-effecting unattended jobs follow approval policy.
- Scheduled jobs enforce model, token, cost, time, network, and execution budgets.
- Job cancellation and history work.
- Local API requires authentication and scoped permissions.
- API event streams do not leak secrets.
- API and scheduler actions are audited.

## 18. Editor bridge and optional UI acceptance

- Editor requests map to an authenticated local session/task API.
- File paths are checked against workspace policy.
- Diagnostics and diff previews refer to current file revisions.
- UI clients cannot bypass the permission engine.
- CLI-only operation remains functional when no editor/web/desktop client is installed.

## 19. Release readiness

A release candidate is acceptable only when:
- All P0 acceptance tests pass.
- All claimed P1/P2 features pass their feature-specific tests.
- The supported platform matrix is tested.
- Upgrade and migration tests pass.
- Compatibility baseline and limitations are published.
- Security review has no unresolved critical/high findings.
- Backup/restore and session recovery are tested.
- README and user documentation are current.


## 20. Bun runtime and build acceptance

- Development, scripts, tests, and source execution run on the pinned Bun version.
- A clean machine without Node.js can run each published standalone CLI artifact.
- Windows .exe launches on every declared Windows target.
- Linux and macOS binaries launch on every declared target and libc/architecture combination.
- Compiled builds correctly load external config, Skills, Plugins, MCP configuration, project files, and provider settings.
- Dynamic imports, package assets, SQLite access, subprocesses, and native dependencies are tested in compiled artifacts.
- The release process records target triple, Bun version, artifact checksum, and signing status.
- Unsupported target platforms are not advertised.
- CLI source development and release artifacts do not require users to install Node.js.


## 21. BITE terminal UI acceptance

- Header identifies workspace, session, provider/model, mode, effort, and permission mode without exposing secrets.
- Wide, medium, narrow, and short terminal layouts are usable.
- Conversation, task, activity, files, agents, jobs, and composer views use consistent state.
- Thinking indicator is based on public activity events and never renders hidden chain-of-thought.
- Task status and progress come from the Task Manager, not assistant text.
- "Agent claims complete" and "runtime verified complete" are distinct.
- Tool cards show real start, finish, error, exit code, and duration events.
- Approval view shows exact command/tool, working directory, target, risk, and grant scope.
- Context usage labels provider-reported and estimated numbers separately.
- Auto-compaction state and result are visible.
- Message queue and interrupt actions are distinct.
- Keyboard navigation, resize, paste, scroll, focus, and shutdown work on the supported terminal matrix.
- Esc follows the documented contextual priority: dismisses popups/overlays first, leaves approval operations blocked when dismissed, requests cancellation for active turns, and never silently discards the draft or exits.
- Shift+Tab cycles interaction modes in the documented order and does not change permission mode or grant authority.
- Tab completion, Ctrl+Tab/ Ctrl+Shift+Tab focus navigation, composer history, list navigation, modal controls, and queue/job/file shortcuts behave according to the keymap.
- Ctrl+C interrupts foreground work and presents an explicit quit confirmation when idle.
- The effective keymap is discoverable and remappable; unsupported terminal sequences have documented fallbacks.
- Terminal modes are restored after normal exit, Ctrl+C, render failure, and provider/tool failure.
- OpenTUI native assets load in source and standalone binaries on each advertised OS/architecture/libc target.
- Plain line-oriented CLI fallback works when the terminal cannot initialize.

## 22. BYV system prompt acceptance

- Official prompt has a version and source file under prompts/.
- Prompt is assembled as an organization-level behavior layer, not mixed with tool schemas or secrets.
- Provider adapter maps canonical instruction roles correctly for each supported protocol.
- Project instructions and Skills do not override managed security policy.
- Prompt changes pass regression evaluations for task following, safe tool use, coding quality, truthful status, and final reporting.
- Prompt does not request or expose hidden chain-of-thought.
