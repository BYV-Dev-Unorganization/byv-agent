# byv-agent

**Autonomous, model-agnostic AI coding agent.**

byv-agent is a modular coding-agent harness with configurable OpenAI-compatible and Anthropic Messages providers, a tool-execution runtime, project-aware context management, and Claude Code-compatible extension formats.

> **Project status:** Planning / specification. This repository describes the target system; it does not claim that features are implemented.

## Product principles

- **Bun-first runtime:** Bun is the official runtime, package manager, workspace manager, test runner, bundler, and standalone executable compiler.
- **Provider freedom:** configure arbitrary OpenAI-compatible endpoints and native Anthropic Messages endpoints without editing source code.
- **Model agnosticism:** capabilities are discovered or configured per model; unsupported features are never silently pretended to work.
- **Claude Code ecosystem compatibility:** support documented formats and behaviors for Skills, MCP configuration, Plugins, Agents, Hooks, settings, and marketplace metadata where specified.
- **Local-first execution:** CLI and workspace operations work locally, with optional isolated Docker/Podman and remote runners.
- **Portable distribution:** publish standalone Windows .exe, Linux binaries, and macOS binaries. Android APK is a separate native/wrapper application target, not a direct Bun compile target.
- **Safe autonomy:** permissions, approvals, workspace boundaries, resource limits, and auditable execution are part of the design.
- **Verifiable coding:** tasks are complete only after configured checks and explicit completion criteria are evaluated.

## Documentation

| Document | Purpose |
|---|---|
| [Master Plan](docs/MASTER-PLAN.md) | Full product scope, modules, requirements, behavior, and non-goals |
| [Technical Architecture](docs/ARCHITECTURE.md) | Runtime boundaries, subsystem contracts, data flow, storage, and deployment |
| [Provider & Model Specification](docs/PROVIDERS.md) | OpenAI-compatible, Anthropic Messages, custom endpoints, capability mapping, routing |
| [Claude Code Compatibility](docs/CLAUDE-CODE-COMPATIBILITY.md) | Skills, MCP, Plugins, Hooks, Agents, settings, and compatibility boundaries |
| [Security Specification](docs/SECURITY.md) | Permissions, sandboxing, secrets, untrusted content, and threat controls |
| [Roadmap](docs/ROADMAP.md) | Dependency-ordered phases and deliverables; intentionally no calendar/day estimates |
| [Acceptance Criteria](docs/ACCEPTANCE-CRITERIA.md) | Testable completion gates for subsystems and release readiness |
| [Configuration](docs/CONFIGURATION.md) | Configuration files, scope precedence, provider settings, secrets, and migrations |
| [CLI Reference](docs/CLI-REFERENCE.md) | CLI command tree, flags, slash commands, status line, permissions, and interactive UX |
| [Test Strategy](docs/TEST-STRATEGY.md) | Unit, contract, integration, end-to-end, security, compatibility, and coding evaluations |
| [Build & Distribution](docs/BUILD-AND-DISTRIBUTION.md) | Bun setup, standalone executable targets, packaging, Android strategy, and release artifacts |
| [Decisions](docs/DECISIONS.md) | Architectural decisions and rationale |

## Initial implementation principles

1. Bun is the required application runtime; users do not need Node.js installed to run a released standalone binary.
2. Keep provider-specific request/response logic inside provider adapters.
3. Normalize provider streams into a documented internal event protocol without discarding provider-specific metadata.
4. Keep the agent loop independent from providers, terminal execution, and extension loaders.
5. Put every tool behind a registry, schema validation, permission checks, timeout/cancellation, and audit logging.
6. Preserve original transcripts and checkpoints when compacting context; compaction changes the active context, not historical records.
7. Treat Claude Code compatibility as a tested compatibility surface, not merely a directory-name convention.
8. Do not run destructive commands or grant broad filesystem/network access by default.
9. Do not invent unsupported model capabilities; expose capability limitations to the user.

## Status

This repository currently contains planning and specification documents. Implementation status must be tracked separately from planned scope. Roadmap phases have completion gates, not time estimates.
