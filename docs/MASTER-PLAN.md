# byv-agent — Complete Master Plan

**Product:** Autonomous AI Coding Agent  
**Document status:** Product and engineering specification  
**Planning policy:** This plan intentionally contains no day-based or calendar-duration estimates.

## 1. Product vision

byv-agent is a local-first, model-agnostic AI coding agent and extensible agent harness. It should understand an existing codebase, plan changes, edit files, execute tools, run tests, debug failures, review diffs, and verify completion. Its extension ecosystem should consume documented Claude Code-compatible formats for Skills, MCP configuration, Plugins, Agents, Hooks, settings, and marketplace metadata wherever the public formats permit.

The project is not merely a chat UI around an LLM. It is a controlled execution system composed of a provider layer, agent runtime, context manager, tools, extension loaders, persistence, security policy, and user interface.

## 2. Product requirements and principles

### 2.1 User requirements

- Users can configure arbitrary OpenAI-compatible providers, including custom base URLs, models, headers, and request options.
- Users can configure native Anthropic Messages API providers.
- Provider credentials are supplied through environment variables or a secure local secret store, not committed project files.
- Users can switch models during a session and assign different models to tasks or subagents.
- Users can set reasoning effort and other provider-specific reasoning controls when the selected model supports them.
- Users can use Claude Code Skills, MCP configuration, Plugins, Agents, Hooks, and marketplace structures with minimal or no file conversion when those formats are publicly documented.
- Agent behavior can be constrained through permissions, approval prompts, sandboxing, budgets, and explicit stop conditions.
- Sessions can be persisted, resumed, inspected, compacted, branched, and recovered.
- The project can begin as a local CLI and later use containerized or remote execution runners without redesigning the agent core.

### 2.2 Non-negotiable design principles

1. **Provider independence:** provider protocol details stay inside adapters.
2. **Capability honesty:** unsupported features are reported as unsupported; they are never silently simulated.
3. **Canonical internal protocol:** provider-specific messages and streams are normalized without discarding useful metadata.
4. **Execution boundary:** the model proposes tool calls; the runtime validates, authorizes, executes, and records them.
5. **Compatibility by specification:** compatibility means parser, loader, configuration, invocation, runtime semantics, and conformance tests—not just matching directory names.
6. **Context durability:** compaction changes active context but never destroys the raw transcript or durable task records.
7. **Safe defaults:** no unrestricted shell, filesystem, network, or destructive operation by default.
8. **Verifiable completion:** a model saying “done” is not sufficient; configured completion criteria and checks must be evaluated.
9. **Local-first, remote-capable:** local workflows remain functional without a cloud control plane.
10. **Observable behavior:** requests, tool calls, retries, compaction, approvals, and failures are inspectable without leaking secrets.

## 3. Product scope

### 3.1 Included

- Interactive CLI and terminal UI.
- OpenAI-compatible and Anthropic Messages provider adapters.
- User-defined provider endpoints and model catalog.
- Model capability registry, routing, fallback, effort controls, token and cost tracking.
- Agent loop, planning, task goals, tool execution, cancellation, retries, continuation.
- Workspace-aware filesystem, terminal, Git, test/build tools.
- Context retrieval, token budgeting, automatic and manual compaction.
- Sessions, transcript storage, checkpoints, rewind, branching, memory.
- Claude Code-compatible Skills, MCP, Plugins, Agents, Hooks, settings, and marketplace formats.
- Subagents, delegated tasks, parallel work, isolated worktrees.
- Code intelligence via search, Tree-sitter, and LSP.
- Background processes, message queue, task notifications.
- Browser automation as an optional tool provider.
- Security policy, permissions, approvals, sandboxing, audit logs.
- Diagnostics, logs, usage reporting, evaluations, release packaging.
- Extensible local, container, VPS, and cloud runner interfaces.

### 3.2 Explicitly deferred or optional

These are not required for the first usable CLI release, but the architecture must not prevent them:

- Native desktop GUI.
- Hosted SaaS control plane and multi-tenant billing.
- Mobile application.
- Voice conversation.
- Public hosted plugin marketplace operated by BYV.
- Distributed agent teams across multiple machines.
- Vector database as a mandatory dependency.
- Autonomous deployment to production.
- Self-modifying core code or unrestricted self-improvement.

## 4. Subsystem inventory

The implementation is organized into the following subsystems. Each subsystem must have its own interface, configuration, tests, logs, and acceptance criteria.

### 01 — Core Agent Engine (P0)

Responsibilities:
- Accept user prompts and session state.
- Build a request from trusted instructions, selected model, active context, and available tools.
- Run the agent loop: request, stream, parse response, authorize tool calls, execute, append results, continue, and finish.
- Support text-only responses, one tool call, multiple tool calls, tool errors, provider errors, partial outputs, and cancellation.
- Enforce iteration, time, token, cost, and tool-call budgets.
- Maintain task state and stop conditions.
- Support plan, normal, debug, review, and autonomous task modes.
- Produce a final summary with files changed, checks performed, unresolved issues, and relevant usage.

Required behavior:
- Model output is never executed directly.
- Tool input is schema-validated before authorization.
- Tool results are represented as typed internal events.
- Retry behavior must be bounded and idempotency-aware.
- Cancellation must propagate to provider streams, child processes, MCP calls when possible, and subagents.
- The core must be independent of any single provider SDK.

### 02 — Provider and Model Management (P0)

Responsibilities:
- OpenAI-compatible adapter.
- Native Anthropic Messages adapter.
- Custom endpoint configuration.
- Provider and model registry.
- API key and custom-header resolution.
- Model capability metadata.
- Model aliases and profiles.
- Streaming normalization.
- Tool-call translation.
- Rate-limit and error normalization.
- Provider health checks and ordered fallback.
- Provider-specific request options that survive round trips.

The detailed provider contract is in [Provider & Model Specification](PROVIDERS.md).

### 03 — Effort and Reasoning Engine (P0)

Responsibilities:
- User-visible effort levels: auto, low, medium, high, xhigh, max, subject to each provider/model capability.
- Global, per-provider, per-model, per-agent, per-task, and per-turn overrides.
- Map canonical effort into native provider parameters.
- Support provider-specific thinking/reasoning controls separately from the canonical effort concept.
- Display the effective level and whether it was native, mapped, ignored, or unsupported.
- Respect model-specific valid values and reject invalid combinations.
- Route tasks by configured role: fast, coding, reasoning, reviewer, summarizer, or custom profile.
- Never claim a model is using a higher reasoning level when the API has no corresponding control.

### 04 — Tool Execution Engine (P0)

Responsibilities:
- Tool registry and discovery.
- JSON Schema or equivalent input validation.
- Permission metadata and risk classification.
- Execution adapter interface.
- Timeout, cancellation, output limits, and error normalization.
- Tool call IDs and result correlation.
- Tool result truncation with durable full-output storage.
- Duplicate-call and retry safety.
- Built-in, MCP, plugin, and future remote tools through one registry.

Built-in tool groups:
- Filesystem: read, write, edit, create directory, list, search, move, copy, delete, metadata.
- Terminal: execute command, persistent session, process list, terminate, stdout/stderr streaming.
- Git: status, diff, log, branch, checkout, stash, commit, worktree, restore, conflict inspection.
- Project: discover, inspect package metadata, list scripts, run tests, run build, diagnostics.
- Context: search files, search symbols, retrieve relevant snippets.
- Optional browser, image, and remote execution tools.

### 05 — Context Intelligence Engine (P0)

Responsibilities:
- Discover repository root and workspace boundaries.
- Load global, project, directory-scoped, and user-approved instructions.
- Support CLAUDE.md and AGENTS.md.
- Support documented project rules and Claude Code settings where compatible.
- Respect .gitignore and configured ignore patterns.
- Search with ripgrep and indexed text search.
- Discover relevant files and symbols.
- Track context provenance: which instruction, message, file, tool result, skill, or MCP result contributed content.
- Estimate tokens per context segment and reconcile with provider usage when available.
- Reserve output budget and safety margin.
- Avoid loading entire repositories by default.
- Track current task, plan, decisions, modified files, test status, pending steps, and known failures as structured durable state.

Context segments to account for:
- Core system/developer instructions.
- User request and recent conversation.
- Project instructions and rules.
- Skill metadata and loaded skill bodies.
- Agent definitions.
- MCP and plugin tool schemas/descriptions.
- Retrieved code and documentation.
- Tool outputs.
- Task plan and goal state.
- Compaction summary.
- Provider-specific metadata that must be retained for continuation.

### 06 — Context Compaction Engine (P0)

Responsibilities:
- Automatic context compaction.
- Manual compaction.
- Configurable threshold by percentage or token count.
- Per-model and per-agent thresholds.
- Target context budget after compaction.
- Preserve recent turns and active tool-call/result pairing.
- Preserve system and developer instructions, user intent, active goal, plan, decisions, constraints, modified-file list, test results, unresolved failures, and pending actions.
- Summarize or prune old conversation and large tool outputs.
- Keep raw transcript and full tool output in persistent storage.
- Rebuild a valid provider message sequence after compaction.
- Prevent compaction in the middle of an incomplete tool-call protocol exchange.
- Retry/fallback when a summarization call fails.
- Emit PreCompact and PostCompact lifecycle events and compatible hook payloads.
- Record trigger reason, old/new token estimates, retained items, removed items, model, and summary metadata.
- Support compaction preview and user-configured protected content.
- Support idle compaction as an optional policy, with clear distinction from task execution and session shutdown.
- Support context reset only with explicit user intent.

Compaction must be a transactional operation:
1. Capture a stable transcript/version.
2. Select eligible history and outputs.
3. Build a structured summary.
4. Validate required task facts and tool protocol consistency.
5. Persist the summary and compaction record.
6. Atomically switch the active context pointer.
7. Resume the agent.
If any step fails, keep the previous active context unchanged.

### 07 — Session and Memory Engine (P0)

Session features:
- Create, list, rename, resume, continue, clear, fork, branch, export, import, archive, and delete sessions.
- Stable session IDs and project association.
- Raw append-only transcript.
- Active compacted context snapshots.
- Session titles, metadata, and timestamps.
- Recovery after process termination.
- Per-session model, effort, mode, budget, permission, and worktree metadata.

Memory features:
- Session memory.
- Project memory.
- User preferences.
- Technical/project architecture notes.
- Subagent memory with user/project/local scopes.
- Searchable memory and provenance.
- Explicit inspect, edit, disable, and delete controls.
- Do not store secrets, credentials, or sensitive data by default.
- Memory retrieval is advisory; project instructions and current user input retain higher authority.

### 08 — Token, Usage, and Cost Management (P0/P1)

Track when available:
- Input tokens.
- Output tokens.
- Cached input/read tokens.
- Cache creation/write tokens.
- Reasoning/thinking tokens.
- Image or multimodal usage where provider reports it.
- Tool-call counts.
- Per-request, per-turn, per-session, per-task, per-agent, per-model, and per-provider usage.
- Estimated versus provider-reported usage.
- Price catalog with effective dates and editable custom rates.
- Cost budgets and stop/warn thresholds.
- Exportable usage report.
- Clear indication when a provider omits usage or pricing is unknown.

### 09 — Skills Compatibility (P0)

Responsibilities:
- Discover user, project, additional-directory, and plugin Skills.
- Parse SKILL.md YAML frontmatter only when the delimiter is at the file start.
- Validate and preserve recognized metadata.
- Support lazy loading of descriptions and full skill body.
- Support manual slash invocation and model-triggered invocation controls.
- Support argument substitution and supporting files.
- Resolve namespace and same-name conflicts deterministically.
- Support skill-specific tool allow/deny metadata without bypassing global permission policy.
- Support dynamic reload where safe.
- Expose skill diagnostics.
- See [Claude Code Compatibility](CLAUDE-CODE-COMPATIBILITY.md).

### 10 — MCP Compatibility (P0)

Responsibilities:
- Parse .mcp.json and supported user/project/local/plugin configuration sources.
- Support stdio, Streamable HTTP, legacy HTTP+SSE compatibility, and WebSocket only where the target public Claude Code config supports it.
- Discover and invoke MCP tools.
- Support MCP resources and prompts.
- Handle server lifecycle, reconnection, timeouts, authentication, headers, OAuth where supported by the chosen SDK/protocol, and dynamic tool list changes.
- Scope and approve server access.
- Avoid exposing credentials in logs or model-visible descriptions.
- Namespace and resolve duplicate server names predictably.

### 11 — Plugin and Marketplace Compatibility (P1)

Responsibilities:
- Load plugin directories and .claude-plugin/plugin.json.
- Support documented components: Skills, Agents, Hooks, MCP server configuration, commands, and other documented component types as they are specified.
- Parse marketplace.json and install sources.
- Support local path and Git-based sources.
- Install, update, enable, disable, remove, inspect, validate, and diagnose plugins.
- Handle version constraints and dependencies where specified.
- Preserve plugin namespace behavior.
- Track trust and permissions for plugin code.
- Never execute plugin scripts merely because the plugin was discovered.
- Support plugin reload and rollback.
- See [Claude Code Compatibility](CLAUDE-CODE-COMPATIBILITY.md).

### 12 — Hooks Lifecycle Engine (P1)

Responsibilities:
- Parse settings and plugin hook configuration.
- Match event, tool, matcher, and pattern.
- Run command, HTTP, prompt, agent, and MCP-tool hook types where compatible and safely implementable.
- Provide documented stdin JSON and output semantics.
- Enforce timeout, cancellation, output limits, and permission policy.
- Support blocking/allow/deny/modify decisions only when the event contract defines them.
- Emit diagnostics for unsupported hook types or fields.
- Implement lifecycle conformance tests.

### 13 — Agent and Subagent Manager (P1)

Responsibilities:
- Parse Claude-compatible agent Markdown definitions.
- Built-in agents: planner, explorer, coder, debugger, tester, reviewer, architect, documentation.
- Per-agent model, effort, tools, permissions, skills, MCP scope, memory, and context budget.
- Task delegation and structured results.
- Parallel execution with concurrency limits.
- Agent lifecycle and cancellation.
- Agent status and progress.
- Subagent isolated context and compaction.
- Optional Git worktree isolation.
- Conflict detection and shared-task coordination.
- Agent teams are an advanced extension; do not claim full Claude Agent Teams compatibility until peer messaging, task coordination, team shutdown, and lifecycle semantics pass conformance tests.

### 14 — Task and Goal Manager (P1)

Responsibilities:
- Convert user requests into task goals and acceptance criteria.
- Maintain task checklist, dependencies, blockers, and status.
- Store task state independently from compactable chat history.
- Verify completion using commands, tests, builds, or explicit checks.
- Resume unfinished tasks.
- Stop on approval requirements, budget limits, repeated failure, or user cancellation.
- Support task plans and goal inspection.

### 15 — Terminal and Filesystem Runtime (P0)

Responsibilities:
- Cross-platform shell discovery: Bash, Zsh, Fish, PowerShell, and CMD as feasible.
- Persistent and one-shot commands.
- Working directory and environment control.
- Live stdout/stderr streaming.
- Exit code, timeout, process tree, signal, and cancellation handling.
- Large-output truncation plus full-output artifact storage.
- Workspace path normalization and symlink checks.
- Text/binary and encoding detection.
- Atomic writes where possible.
- File snapshot or change journal integration.
- Respect ignore rules and permission policy.

### 16 — Git and Checkpoint Manager (P1)

Responsibilities:
- Git repository detection and status/diff/log/branch operations.
- Safe staging and commit workflows.
- Branch and worktree creation.
- Agent-specific worktrees.
- Conflict detection and explanation.
- Per-turn file snapshots or durable change journal.
- Undo/rewind/restore UI.
- Checkpoint listing and metadata.
- Restore only selected changes where possible.
- Never silently discard user changes.
- Require explicit approval for reset, clean, force push, destructive checkout, and similar operations.

### 17 — Code Intelligence Engine (P1)

Responsibilities:
- Language detection.
- Tree-sitter syntax parsing.
- LSP client integration.
- Diagnostics, hover, definition, references, symbols, rename, and code actions where server supports them.
- Symbol and dependency index.
- Cross-file references.
- Changed-file diagnostics.
- Server installation/configuration guidance.
- Language priority: TypeScript/JavaScript and Python first; Go, Rust, Java, PHP, C/C++, C#, Ruby and others through LSP.

### 18 — Browser Automation (P2)

Responsibilities:
- Search and page reading via configured providers.
- Playwright browser control.
- DOM inspection and interaction.
- Screenshot capture and visual verification.
- Authenticated browsing only through user-authorized sessions.
- Download/upload handling with approval.
- Separate browser contexts per task.
- Prompt-injection handling for page content.
- No credential extraction or unauthorized access.

### 19 — Background and Queue Manager (P1)

Responsibilities:
- Background shell processes.
- Long-running jobs.
- Background subagents.
- Persistent task queue.
- Queue new user messages while agent is working.
- Immediate steering/interrupt.
- Pause, resume, cancel, and clear queue.
- Progress, logs, notifications, and stall detection.
- Recover task metadata after restart.
- Ensure only one authoritative writer per shared workspace unless explicit isolation exists.

### 20 — Security and Permission Engine (P0)

Responsibilities:
- Read/write/execute/network/destructive permission categories.
- Per-tool and per-path policies.
- Ask, allow, deny, and scoped temporary approval.
- Workspace and symlink boundary enforcement.
- Secret redaction and credential references.
- Prompt-injection defenses and untrusted-content labels.
- MCP and plugin trust decisions.
- Hook and plugin code approval.
- Audit log.
- Sandbox policy.
- Resource limits.
- Safe defaults.
- See [Security Specification](SECURITY.md).

### 21 — Sandbox and Remote Runtime (P2)

Responsibilities:
- Local process runner.
- Docker/Podman runner.
- Remote runner interface for VPS/cloud.
- Worker registration and authentication.
- Per-task isolated workspace.
- Restricted network and filesystem.
- CPU, memory, disk, and process limits.
- Short-lived credentials.
- Artifact and log transfer.
- Task status, heartbeat, cancellation, and recovery.
- Execution policy consistent across local and remote runners.

### 22 — CLI and Interactive UI (P0)

Responsibilities:
- CLI commands and flags.
- Streaming text and tool status.
- Multiline input editor.
- Prompt and command history.
- Slash command parser and autocomplete.
- Skill and agent autocomplete.
- Model/provider/effort selector.
- Permission confirmation UI.
- Plan approval UI.
- Message queue UI.
- Context and token usage status.
- Cost and task status display.
- Diff and checkpoint viewer.
- Background process monitor.
- Theme and keyboard shortcuts.
- Image input where terminal capabilities permit.
- Non-interactive mode for automation.

### 23 — Configuration and Diagnostics (P0)

Responsibilities:
- Layered settings: managed, CLI/session, project, local-project, user, defaults, with documented precedence.
- JSON/schema validation.
- Environment variable expansion with secret-safe handling.
- Config migrations and versioning.
- Provider/model validation.
- Extension diagnostics.
- byv doctor command.
- Effective configuration report without secrets.
- Debug mode and redacted logs.
- Feature/capability status.

### 24 — Observability and Evaluation (P0/P1)

Responsibilities:
- Structured logs.
- Trace IDs for sessions, turns, provider calls, tool calls, hooks, MCP calls, and subagents.
- Token/cost/latency metrics.
- Error categorization.
- Optional OpenTelemetry export.
- Local diagnostic bundles with secret redaction.
- Deterministic unit and integration suites.
- Compatibility fixtures.
- Coding task evaluation set.
- Regression tracking across model/provider changes.
- User opt-in for telemetry; telemetry off by default unless user explicitly enables it.

### 25 — Scheduler and Automation Engine (P2)

Responsibilities:
- One-time and recurring scheduled tasks.
- Cron-like schedule parsing with timezone awareness.
- Session-scoped scheduled reminders/actions where supported.
- Event-triggered jobs from hooks, webhooks, repository state, or user-defined triggers.
- Job queue, concurrency limits, deduplication, idempotency keys, and retry policy.
- Run policy for whether tasks may execute unattended.
- Approval requirements for side-effecting scheduled jobs.
- Job history, status, logs, cancellation, and missed-run handling.
- Explicit resource, token, cost, network, and execution budgets.
- Prevent overlapping runs unless concurrency is explicitly allowed.

Scheduled execution must not bypass the normal permissions, sandbox, secrets, and audit systems.

### 26 — Integration API and Editor Bridge (P2/P3)

Responsibilities:
- Local authenticated API for session/task inspection and approved control operations.
- WebSocket or equivalent event stream for status and logs.
- Editor/IDE integration interface for diagnostics, file references, diff previews, and task handoff.
- Optional web dashboard or desktop interface.
- External automation integrations through explicit authentication and permission scopes.
- Stable versioned API and backward-compatibility policy.

The CLI remains first-class; no hosted SaaS or desktop client is required for the initial release.

## 5. Canonical user experience

### 5.1 Core modes

- **Normal:** direct agent loop for bounded tasks.
- **Plan:** inspect and produce a plan before edits; require approval according to policy.
- **Auto:** continue through a task goal while respecting budget and permissions.
- **Debug:** investigate failures, identify root cause, apply a bounded fix, run regression checks.
- **Review:** inspect changes and report actionable findings; read-only by default.
- **Ask:** answer using project context without modifying files.
- **Shell:** direct user-controlled shell access, clearly separated from model-authorized execution.

### 5.2 Essential slash commands

- /help, /clear, /exit, /quit
- /model, /provider, /effort
- /plan, /normal, /auto, /ask, /debug, /review
- /compact, /context, /autocompact
- /usage, /cost, /stats
- /permissions, /approve, /deny
- /skills, /agents, /mcp, /plugin, /hooks
- /tasks, /goal, /queue, /background
- /undo, /rewind, /checkpoints, /diff
- /resume, /sessions, /branch, /export
- /doctor, /config, /status

Command names are the proposed byv-agent UX. Compatibility aliases may be added, but a byv command must not falsely claim identical Claude Code behavior unless tested.

### 5.3 CLI commands

- byv
- byv init
- byv doctor
- byv config
- byv providers
- byv models
- byv skills
- byv mcp
- byv plugin
- byv agents
- byv sessions
- byv run
- byv resume
- byv version

## 6. Configuration and directory layout

Global internal data:
- ~/.byv-agent/config/
- ~/.byv-agent/providers/
- ~/.byv-agent/sessions/
- ~/.byv-agent/memory/
- ~/.byv-agent/logs/
- ~/.byv-agent/cache/
- ~/.byv-agent/plugins/
- ~/.byv-agent/marketplaces/
- ~/.byv-agent/checkpoints/
- ~/.byv-agent/state/

Project compatibility files:
- CLAUDE.md
- AGENTS.md
- .claude/settings.json
- .claude/settings.local.json (must be ignored by Git)
- .claude/rules/
- .claude/skills/
- .claude/agents/
- .claude/commands/
- .mcp.json
- .claude-plugin/plugin.json inside plugin directories

byv-agent-owned project state may live in .byv-agent/ only when it is safe and useful to share. Secrets, session transcripts, local overrides, and machine-specific state must not be committed.

## 7. Recommended technology stack

- Language: TypeScript with strict type checking.
- Runtime: supported Node.js LTS release.
- Monorepo: pnpm workspaces.
- CLI parsing: Commander or citty; select one after a small technical spike.
- TUI: Ink and a prompt library where needed.
- Validation: Zod or JSON Schema with a single canonical schema strategy.
- HTTP: native fetch or a thin typed client.
- Provider SDKs: use official SDKs where they preserve required protocol features; retain raw HTTP adapter escape hatches for compatible gateways.
- MCP: official TypeScript SDK.
- Processes: Execa or carefully wrapped Node child_process.
- Persistence: SQLite with migrations; Drizzle is an optional query layer.
- Search: ripgrep plus SQLite FTS5.
- Syntax: Tree-sitter.
- Language intelligence: LSP client.
- Testing: Vitest; Playwright for browser/e2e.
- Logging: Pino.
- Tracing: OpenTelemetry optional export.
- Containers: Docker/Podman.
- CI: GitHub Actions.

Avoid using multiple competing libraries for the same subsystem without a documented reason. The exact dependency versions must be selected and pinned during implementation, not guessed in this planning document.

## 8. Release gates

A milestone is complete only when:
- Its documented behavior is implemented.
- Unit and integration tests pass.
- Failure and cancellation paths are tested.
- Security policy is enforced.
- User-visible diagnostics exist.
- Relevant compatibility fixtures pass.
- Documentation and examples are updated.
- No known critical data-loss or secret-leak issue remains.

See [Roadmap](ROADMAP.md) and [Acceptance Criteria](ACCEPTANCE-CRITERIA.md).

## 9. Out of scope for the initial product

- A proprietary LLM foundation model.
- Automatic unrestricted production deployment.
- Silent destructive actions.
- Mandatory cloud account or central SaaS service.
- Mandatory vector database.
- Claiming full Claude Code parity without public-format conformance and behavioral tests.

## 10. Official reference documents

The compatibility implementation must track the current public documentation and record the tested compatibility baseline in release notes:

- Claude Code Skills: https://code.claude.com/docs/en/skills
- Claude Code Plugins: https://code.claude.com/docs/en/plugins
- Claude Code Hooks: https://code.claude.com/docs/en/hooks
- Claude Code Subagents: https://code.claude.com/docs/en/sub-agents
- Claude Code MCP: https://code.claude.com/docs/en/mcp
- Claude Code Settings: https://code.claude.com/docs/en/settings
- Anthropic Messages API: https://platform.claude.com/docs/en/api/http/messages
- OpenAI API text and Responses API: https://developers.openai.com/api/docs/guides/text
- MCP specification: https://modelcontextprotocol.io/specification/latest
