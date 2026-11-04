# byv-agent Architecture Decision Records

This file records baseline decisions. A decision can be superseded only through a reviewed change that explains migration and compatibility impact.

## ADR-001 — TypeScript and Bun as the required runtime

**Decision:** Use TypeScript with strict checking on Bun. Bun is the application runtime, package manager, workspace manager, test runner, bundler, and standalone executable compiler. Node.js is not a production runtime requirement.

**Rationale:** Bun provides an integrated TypeScript/JavaScript runtime and can compile applications into standalone executables for documented Windows, Linux, and macOS targets.

**Consequences:** Pin the Bun version. Test dependencies and Node compatibility APIs on Bun. Validate native addons per target. Keep the release matrix limited to supported desktop/server executable targets.

## ADR-002 — Build the core agent loop in byv-agent

**Decision:** Own the canonical agent loop, state machine, tool registry, and provider contracts rather than making a third-party agent framework the mandatory runtime.

**Rationale:** Provider flexibility and Claude Code-compatible extension behavior require predictable control over tool calls, context, permissions, cancellation, and compaction.

**Consequences:** More core engineering responsibility; external libraries may still be used behind interfaces.

## ADR-003 — Two first-class provider protocols

**Decision:** Support OpenAI-compatible Chat Completions and native Anthropic Messages as first-class adapters. Add native OpenAI Responses as a distinct adapter where needed.

**Rationale:** User requires OpenAI-compatible and Anthropic Messages; protocol distinctions should be explicit.

**Consequences:** Provider conformance fixtures are required. “OpenAI-compatible” does not imply feature parity across gateways.

## ADR-004 — Canonical internal message and event model

**Decision:** Normalize provider-specific messages and streams into an internal typed protocol.

**Rationale:** The agent loop must not depend on a single provider format.

**Consequences:** Adapters must preserve structured content, tool IDs, usage, and provider-specific metadata needed for continuation.

## ADR-005 — Claude Code public-format compatibility

**Decision:** Consume documented Claude Code formats for Skills, MCP configuration, Plugins, Agents, Hooks, settings, and marketplace metadata.

**Rationale:** User wants to reuse the Claude Code extension ecosystem.

**Consequences:** Compatibility is versioned and tested; proprietary or undocumented runtime behavior is not promised.

## ADR-006 — SQLite for initial persistent state

**Decision:** Use SQLite with migrations for sessions, transcripts, task state, usage, checkpoints, extension registry, and audit metadata.

**Rationale:** Local-first operation, transactional semantics, easy backup, and low operational complexity.

**Consequences:** Large artifacts should be stored separately with references; secret values must not be stored in plaintext.

## ADR-007 — Raw transcript is immutable history; compacted context is a projection

**Decision:** Keep raw transcript and artifacts, while active context snapshots can be compacted.

**Rationale:** Auto-compact must not destroy recoverability or auditability.

**Consequences:** Compaction requires transactional snapshot switching, retention policy, and recovery tests.

## ADR-008 — Security is a runtime gate

**Decision:** All tools, plugins, hooks, MCP calls, filesystem operations, and process execution pass through the permission/security layer.

**Rationale:** Prompt text is not an enforcement boundary.

**Consequences:** No extension may bypass global deny rules; default permissions must remain restrictive.

## ADR-009 — Local-first with optional runner abstraction

**Decision:** The first-class execution path is local CLI, but tools target a runner interface supporting containers and remote workers.

**Rationale:** Avoid requiring a cloud control plane while preserving future hybrid execution.

**Consequences:** Workspace snapshots, task identities, credentials, cancellation, and artifacts need stable contracts.

## ADR-010 — Feature claims require acceptance evidence

**Decision:** Mark features supported only after tests and documented compatibility criteria pass.

**Rationale:** Prevent misleading claims, especially around Claude Code compatibility and heterogeneous provider capabilities.

**Consequences:** Release notes and diagnostics must expose partial/unsupported behavior.


## ADR-011 — Standalone Bun executable distribution

**Decision:** Ship byv-agent CLI as Bun standalone executables for supported desktop and server operating systems: Windows, Linux, and macOS.

**Rationale:** Bun compile embeds the runtime and supports standalone executable targets across the selected desktop/server platforms.

**Consequences:** Release automation builds and tests each target binary independently.


## ADR-012 — OpenTUI Core for the first-class terminal experience

**Decision:** Use OpenTUI Core as the initial renderer for BITE. The UI is event-driven and separate from provider/tool execution. @opentui/react is optional and requires a Bun/native/compiled-target validation before adoption.

**Rationale:** OpenTUI is designed as a native terminal UI framework, supports Bun directly, and provides the layout, input, scrolling, Markdown, code, diff, and embedded-terminal building blocks needed for a coding-agent workspace.

**Consequences:** Pin a tested OpenTUI version. Run a compatibility spike and verify FFI, native assets, and compiled executable behavior on every advertised platform. Keep a line-oriented CLI fallback.

## ADR-013 — BYV organization prompt is versioned separately from runtime policy

**Decision:** Maintain the default organization-level behavior prompt at prompts/BYV-DEV-SYSTEM-PROMPT.md. Keep it separate from machine-enforced permission and security policy.

**Rationale:** BYV Dev Unorganization needs its own product voice, engineering workflow, provider-neutral behavior, and truthful progress/reporting standards.

**Consequences:** Prompt changes require versioning and regression evaluation. The prompt cannot grant tool permissions or override managed security controls.
