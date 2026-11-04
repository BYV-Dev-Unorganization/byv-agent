# byv-agent Technical Architecture

## 1. Architecture goals

- Keep the agent runtime independent from provider protocol, terminal implementation, storage engine, and extension format.
- Make every effectful operation pass through the tool and permission layers.
- Support streaming, cancellation, resumption, compacted context, and durable background tasks.
- Use explicit subsystem contracts and typed events.
- Keep local CLI execution as the first-class path while defining stable interfaces for container and remote runners.

## 2. Runtime foundation: Bun

Bun is the required runtime for source execution and the official build toolchain.

Runtime requirements:
- Use Bun APIs for first-class runtime operations where they provide a clear benefit: process execution (`Bun.spawn`), file access (`Bun.file`), SQLite (`bun:sqlite`), and build/compile.
- Use Web-standard APIs (fetch, Request, Response, AbortSignal, streams) where practical for portable provider and HTTP code.
- Node.js compatibility APIs may be used when necessary, but every such dependency must pass Bun runtime and standalone compile tests.
- Avoid native Node-API addons unless each supported target is verified.
- Avoid assuming that a source-level dependency will be bundled correctly; test dynamic imports, package assets, subprocess launch, and plugin loading in compiled builds.
- Bun is not assumed to exist on end-user machines when distributing a standalone CLI.

The agent core must not depend on a Node.js executable or npm/yarn/pnpm being installed on the target machine. Released standalone CLI binaries embed Bun; they do not embed Git, Docker, browser engines, language servers, or arbitrary system tools.

## 3. Logical topology

    User / CLI / Automation API
              |
       Interaction Layer
              |
       Session Controller
              |
        Core Orchestrator
          /    |     \
     Context  Model  Task/Goal
     Engine  Router  Manager
          \    |     /
         Canonical Agent Protocol
                  |
             Tool Registry
                  |
          Permission / Policy Gate
                  |
          Execution Dispatcher
        /       |        |       \
    Local FS  Process   MCP    Plugin/Remote
        \       |        |       /
             Result Store
                  |
        Transcript / Session DB
                  |
       Compaction / Memory / Usage

Cross-cutting systems:
- Configuration resolver
- Hooks lifecycle dispatcher
- Security and secret redaction
- Observability and audit
- Background job supervisor

## 4. Runtime boundaries

### 3.1 Interaction layer

Owns terminal rendering, input editing, command parsing, interactive prompts, status line, queue display, and non-interactive command invocation. It must not call providers or tools directly; it sends commands to the session controller.

### 3.2 Session controller

Owns session lifecycle, active model/effort/mode, user steering, queue semantics, interrupt handling, transcript append, session resume, and state recovery.

### 3.3 Core orchestrator

Owns the model/tool loop and task state transitions. It must not perform filesystem or process effects directly. It requests tools through the registry and execution dispatcher.

### 3.4 Provider manager

Owns provider configuration, adapters, capability lookup, request construction, stream parsing, usage extraction, retry classification, routing, and fallback. It returns canonical events.

### 3.5 Context engine

Owns instruction discovery, retrieval, token estimates, context assembly, context provenance, and budget calculation.

### 3.6 Compaction engine

Owns history selection, summary creation, compaction validation, active-context replacement, event emission, and recovery. It does not delete the raw transcript.

### 3.7 Tool execution dispatcher

Owns schema validation, tool ID correlation, permission check, approval request, runner selection, timeouts, cancellation, output capture, and result persistence.

### 3.8 Extension manager

Owns Skills, Plugins, Agents, Hooks, MCP configuration, marketplace sources, extension namespaces, validation, trust state, and reload.

### 3.9 Execution runners

Implement a common runner contract:
- Local process runner.
- Docker/Podman runner.
- Remote worker runner.

The runner receives a task-scoped execution request and returns a typed result, artifacts, logs, exit status, and resource metadata. It must not receive unrestricted host credentials.

## 5. Core protocol types

Define canonical types before implementing adapters. Names below are conceptual and can be refined in code.

### 4.1 Canonical message model

A message has:
- Stable internal ID.
- Role: system, developer, user, assistant, or tool.
- Ordered content blocks.
- Timestamp.
- Session and turn IDs.
- Source/provenance metadata.
- Optional provider-specific metadata in a namespaced extension field.

Content blocks must represent:
- Text.
- Image or other supported multimodal input.
- Tool call with ID, name, and structured arguments.
- Tool result with matching call ID and error state.
- Refusal or safety block.
- Provider reasoning/thinking metadata where the provider exposes it and policy permits storing it.
- Untrusted text boundaries or labels.

Never flatten a tool call/result exchange into ordinary text if the provider protocol requires structured blocks.

### 4.2 Canonical provider event stream

Normalize streams to events such as:
- response.started
- response.text.delta
- response.reasoning.delta (metadata only; do not expose private reasoning by default)
- response.tool_call.started
- response.tool_call.arguments.delta
- response.tool_call.completed
- response.usage
- response.completed
- response.failed
- response.cancelled

Store original provider event metadata for debugging when safe and configured. The adapter is responsible for protocol-specific ordering and final aggregation.

### 4.3 Tool contract

Every tool declares:
- Stable namespaced name.
- Description.
- Input schema.
- Risk category.
- Required permissions.
- Execution adapter.
- Timeout and output budget.
- Cancellation behavior.
- Idempotency classification.
- Whether it is read-only.
- Provenance and owner: built-in, MCP server, plugin, or remote.

Execution sequence:
1. Validate call ID, tool name, and argument schema.
2. Resolve tool and current workspace.
3. Evaluate global, project, and session permissions.
4. Request approval when required.
5. Invoke the selected runner.
6. Stream progress and output.
7. Persist bounded result and full artifact when appropriate.
8. Emit post-tool hooks.
9. Return a correlated tool result to the agent loop.

### 4.4 Capability model

Model capabilities must be explicit and tri-state where appropriate: supported, unsupported, or unknown. Include:
- Streaming.
- Tool calling.
- Parallel tool calling.
- Vision/input modalities.
- Structured output.
- Reasoning/effort controls.
- Thinking blocks.
- Prompt caching.
- Context window.
- Maximum output.
- Usage reporting.
- Token counting endpoint.
- Provider-specific tool extensions.

Unknown is not equivalent to supported.

## 6. Agent state machine

Suggested states:
- idle
- understanding
- planning
- awaiting_plan_approval
- requesting_model
- streaming_model
- awaiting_tool_approval
- executing_tool
- waiting_for_background_job
- compacting
- awaiting_user_input
- completed
- failed
- cancelled

Valid state transitions must be centralized and tested. A terminal state cannot silently re-enter execution. Resume creates a controlled transition from persisted state.

## 7. Agent loop

1. Load session and effective configuration.
2. Resolve project root and trust status.
3. Load system/developer instructions and project instructions.
4. Retrieve task-relevant context.
5. Resolve provider, model, effort, capabilities, and budget.
6. Build canonical request.
7. Stream provider events.
8. Accumulate text and structured tool calls.
9. Validate completed tool calls.
10. Ask permission where required.
11. Execute tools through a runner.
12. Persist tool outputs and artifacts.
13. Run lifecycle hooks.
14. Add correlated tool results.
15. Recalculate token/context budget.
16. Compact if the threshold or provider-specific requirements demand it.
17. Continue until final response, stop condition, budget exhaustion, approval boundary, or cancellation.
18. Verify task goal and configured checks.
19. Persist final summary and usage.

## 8. Effort engine design

Canonical levels: auto, low, medium, high, xhigh, max.

The effort resolver precedence should be:
1. Explicit per-turn override.
2. Explicit task override.
3. Explicit agent override.
4. Model profile override.
5. Provider/model default.
6. Global default.
7. Auto.

This precedence is a byv-agent decision; expose the effective result in /status and diagnostics.

The provider adapter maps effort and thinking separately. Record:
- requested level;
- effective provider parameter;
- whether mapped natively;
- unsupported or downgraded state;
- reason and provider capability.

Do not assume identical semantics among OpenAI-compatible gateways and Anthropic Messages. Gate values by the selected model.

## 9. Context budgeting and compaction design

### 8.1 Context budget

Compute:
- model context limit;
- reserved output budget;
- system/developer instruction budget;
- tool schema budget;
- active conversation budget;
- retrieved context budget;
- safety margin.

Estimate locally before requests, then reconcile with provider-reported usage. Do not present local estimates as provider-exact token usage.

### 8.2 Automatic compaction

Configurable options:
- enabled;
- trigger percentage;
- trigger token count;
- target post-compaction percentage/token count;
- preserve recent turn count;
- protected content selectors;
- compaction model/profile;
- per-provider and per-model override;
- per-agent override;
- minimum remaining budget;
- behavior on compaction failure.

Compaction should happen before context exhaustion, not after a provider rejects the request. It must preserve active tool-call pairing, task state, required instructions, decisions, constraints, file changes, test results, blockers, and pending work.

### 8.3 Compaction transaction

- Freeze a transcript revision.
- Build an eligible history selection.
- Exclude instructions and protected facts.
- Generate a structured summary with sections: current goal, user constraints, decisions, codebase facts, modified files, tests/checks, failures, pending actions, references to source messages/artifacts.
- Validate the summary against required fields.
- Persist summary and source-range metadata.
- Commit new active context snapshot atomically.
- Emit PreCompact and PostCompact lifecycle events.
- Resume with no duplicate tool execution.

If summarization, validation, or persistence fails, retain the previous active context and report the failure.

### 8.4 Full transcript retention

Raw user/assistant messages and tool artifacts remain in persistent history according to retention settings. Compaction is a projection for the active model request, not destructive transcript rewriting.

## 10. Storage architecture

SQLite is the initial durable store. Use migrations and transactions.

Suggested entities:
- sessions
- turns
- messages
- content_blocks
- tool_calls
- tool_results
- artifacts
- context_snapshots
- compaction_runs
- task_goals
- task_items
- checkpoints
- file_snapshots
- providers
- model_profiles
- usage_records
- price_catalog
- approvals
- background_jobs
- agent_runs
- extension_registry
- hook_runs
- audit_events
- memory_entries

Large binary artifacts and full tool outputs may be stored in a content-addressed local artifact directory with references in SQLite.

Secrets must not be stored in plaintext SQLite. Store secure references or use an OS credential manager/encrypted secret store; environment variables are an acceptable initial non-persistent option.

## 11. Configuration resolution

Define and document exact precedence. Recommended conceptual order, highest to lowest:
1. Managed policy that cannot be overridden.
2. Explicit CLI flag or session override.
3. Project local settings.
4. Shared project settings.
5. User settings.
6. Built-in defaults.

Security-deny rules should be able to remain restrictive even when a less-trusted scope attempts to relax them. Lists must declare whether they replace or merge; never rely on accidental JSON merge behavior.

Validate all files and show the effective configuration with secrets redacted.

## 12. Concurrency and multi-agent

- Main agent owns the user-facing session.
- Subagents receive scoped task packets and separate context.
- Each agent has independent model, effort, permissions, memory, budget, and cancellation token.
- Parallel file modifications require isolated worktrees or explicit ownership locks.
- Shared task board stores dependencies and completion evidence.
- Subagent results return structured summaries and artifact references.
- Parent agent verifies integration and runs final tests.
- Limit nested delegation and maximum concurrent agents.

## 13. Remote execution boundary

The control/orchestration process may remain local while execution occurs inside local containers or remote runners. The interface should carry:
- task ID and runner ID;
- workspace snapshot or Git reference;
- command/tool request;
- scoped environment references;
- resource limits;
- network policy;
- cancellation token;
- log/artifact destinations.

Remote workers authenticate with short-lived credentials. Never transmit provider API keys or unrelated user secrets to a worker unless a scoped task explicitly requires them.

## 14. Observability

All major operations should carry session_id, turn_id, task_id, agent_id, provider_request_id, tool_call_id, and trace_id where applicable.

Log:
- provider selected and capability mapping;
- request timing and normalized errors;
- tool validation and permission decision;
- process exit and resource usage;
- compaction trigger/result;
- queue and background state;
- extension load failures;
- session recovery;
- task completion evidence.

Redact secrets, authorization headers, environment values, private reasoning content, and sensitive file contents.

## 15. Testing boundaries

- Unit tests per subsystem.
- Contract tests for provider adapters.
- Golden fixtures for extension parsing.
- Integration tests with fake provider and fake tool runner.
- MCP server fixtures.
- Plugin fixture repositories.
- Cross-platform process tests.
- Compaction round-trip and recovery tests.
- Security tests for path traversal, symlinks, shell injection, prompt injection, and secret leakage.
- End-to-end coding tasks with deterministic repositories.
