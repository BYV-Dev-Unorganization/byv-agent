# Claude Code Ecosystem Compatibility Specification

## 1. Objective and compatibility promise

byv-agent will consume documented Claude Code-compatible extension formats so users can reuse existing project configuration and extensions with minimal or no conversion.

Compatibility is versioned and testable. It does not mean that byv-agent reproduces Anthropic's private system prompt, hosted services, proprietary runtime, or undocumented internal behavior.

Each release must publish:
- the Claude Code documentation baseline reviewed;
- supported formats and fields;
- supported runtime semantics;
- unsupported or partially supported fields;
- conformance fixture results;
- known compatibility differences.

The implementation must preserve unknown metadata when safely possible and issue diagnostics rather than silently dropping fields.

## 2. Compatibility roots and discovery scopes

Support discovery from documented locations where relevant:
- User scope: user-level Claude configuration and extensions.
- Project scope: project-root .claude directory and .mcp.json.
- Local project scope: project-local settings excluded from Git.
- Additional directories explicitly supplied by the user.
- Plugin root and plugin component directories.
- Nested project directories where documented discovery rules apply.

The compatibility resolver must record the source and scope of each discovered component. Define explicit precedence and merge rules per component type; do not assume the same rule for skills, MCP servers, hooks, and settings.

## 3. Instruction files and rules

Support:
- CLAUDE.md as project/user instruction content.
- AGENTS.md as an additional ecosystem instruction file.
- .claude/rules/ as scoped project rules where the documented format is supported.
- Imported instruction files and references only within approved workspace boundaries.

Requirements:
- Resolve instruction files from the actual working directory and ancestor/project scope.
- Support nested directory-specific rules where documented.
- Preserve source path and scope.
- Surface conflicting instructions in diagnostics.
- Never treat instructions loaded from an untrusted repository as security policy.
- Keep policy/permission enforcement outside prompt text.

## 4. Skills

### 4.1 Directory conventions

Support standalone Skills in the documented user and project locations, including:
- ~/.claude/skills/<skill-name>/SKILL.md
- .claude/skills/<skill-name>/SKILL.md
- plugin-root/skills/<skill-name>/SKILL.md

Support Claude Code command files from .claude/commands/ where the command-file format overlaps with Skills. Preserve the distinction where name/path semantics differ.

### 4.2 SKILL.md parsing

- YAML frontmatter must begin at the first line, delimited by ---.
- Parse YAML safely; do not execute YAML tags or custom constructors.
- Markdown after the closing delimiter is the skill body.
- If frontmatter is invalid, report a diagnostic and apply a documented safe fallback; do not silently pretend metadata was read.
- Boolean values should follow accepted documented forms where possible.
- Preserve unknown fields for forward compatibility.

Fields to support and test against the current public reference include:
- name
- description
- when_to_use
- argument-hint
- arguments
- disable-model-invocation
- user-invocable
- allowed-tools
- disallowed-tools
- model
- context
- agent
- paths
- hooks
- license
- compatibility
- metadata

The list may expand as the upstream public format changes. Implement schema/version fixtures rather than assuming the table will never change.

### 4.3 Runtime behavior

- Show eligible Skills in slash-command completion.
- Use namespacing for plugin-provided Skills to avoid collisions.
- Support manual invocation.
- Support model-triggered invocation only when enabled.
- Respect disable-model-invocation and user-invocable independently.
- Support argument substitution, including documented positional and named argument forms.
- Support argument-hint in completion.
- Support allowed-tools and disallowed-tools as scoped declarations, but they must never override byv-agent global deny rules, managed restrictions, or user approval policy.
- Support model override only if the target model exists and policy permits it.
- Support context fork/agent invocation where supported by the byv-agent subagent runtime.
- Load skill body on demand, not all bodies at session start.
- Resolve supporting files relative to the skill directory.
- Expand documented plugin/skill directory variables only in supported locations and without permitting path traversal.
- Support optional scripts, references, and assets as files; execute scripts only through the normal tool permission and sandbox path.
- Support skill hooks only after hook conformance and security gates pass.

### 4.4 Collision and loading rules

- Detect same-name Skills at different scopes.
- Apply an explicit, documented precedence/namespace rule.
- Do not silently hide collisions.
- Plugin Skills use plugin namespace identifiers.
- Cache only with invalidation on file changes.
- Provide /skills list, inspect, reload, validate, and source diagnostics.

## 5. MCP

### 5.1 Configuration files and scope

Support the public Claude Code MCP configuration shape in .mcp.json and relevant documented user/project/plugin sources. The exact scope and precedence must match the compatibility version declared for the release.

Configuration parser requirements:
- Validate JSON and known fields.
- Support environment variable references safely.
- Support command, args, env, URL, headers, and transport type fields where defined.
- Do not display secret values.
- Report invalid entries while allowing unrelated valid servers to load.

### 5.2 Transports

Target:
- stdio.
- Streamable HTTP (including documented type aliases).
- Legacy HTTP+SSE compatibility for existing servers; mark deprecated where upstream does.
- WebSocket only when the relevant documented Claude Code config format and implementation contract are supported.

Remote HTTP authentication, OAuth, headers, reconnection, and refresh behavior must follow the current MCP SDK and documented client semantics. Do not invent OAuth behavior when the selected SDK or server does not provide it.

### 5.3 MCP capabilities

- Initialize and negotiate protocol version/capabilities.
- List and call tools.
- List/read resources.
- List/get prompts.
- Handle server notifications and tool-list changes.
- Reconnect and refresh capabilities when supported.
- Capture structured errors and server diagnostics.
- Scope servers to project/user/plugin/agent as documented.
- Ask before connecting to untrusted project servers where trust policy requires it.
- Filter available tools per agent and permission policy.
- Avoid flooding model context with every remote schema; support tool search or lazy schema exposure where appropriate.

### 5.4 Configuration lifecycle

- Add, remove, enable, disable, inspect, authenticate, and test a server.
- Detect duplicate server names.
- Show effective server source and status.
- Terminate owned stdio child processes on shutdown.
- Apply timeouts, cancellation, output limits, and process resource limits.
- Never pass MCP credentials as model-visible text.

## 6. Plugins and marketplaces

### 6.1 Plugin format

Support documented plugin layout with:
- .claude-plugin/plugin.json manifest.
- skills/
- agents/
- hooks/hooks.json
- .mcp.json
- commands/ where documented.
- other documented plugin components when their behavior is implementable and tested.

Do not assume a component directory must always be listed in plugin.json; follow the public plugin rules for the compatibility baseline.

### 6.2 Manifest

Parse and validate:
- name;
- version;
- description;
- author and other documented metadata;
- component/path metadata and dependencies where the upstream schema supports them.

Retain unknown metadata. Reject unsafe paths and invalid names. Ensure the plugin root cannot escape the installation directory through symlinks or traversal.

### 6.3 Marketplace

Support documented marketplace.json structure and sources. Provide:
- marketplace add/list/inspect/update/remove;
- plugin discovery;
- install/uninstall;
- enable/disable;
- update;
- version pinning where supported;
- source integrity and trust review;
- rollback;
- diagnostics.

Marketplace source can be Git-based or a supported local source. Do not run install scripts without explicit approval and sandboxing.

### 6.4 Plugin namespace and runtime

- Namespace Skills and agents by plugin identity where upstream does.
- Merge hook declarations according to documented behavior.
- Resolve MCP name collisions according to documented precedence.
- Track which tool, skill, agent, and hook came from which plugin.
- Apply plugin permissions separately from installation trust.
- Reload safely; avoid replacing active code during a running hook/tool invocation.
- Support plugin validation against fixtures and example plugins.

### 6.5 Compatibility boundaries

A Claude Code plugin may contain components whose runtime depends on proprietary UI, hosted authentication, or runtime-only behavior unavailable to byv-agent. The loader must report such components explicitly as unsupported/partial and continue loading supported components when safe.

## 7. Agents and subagents

### 7.1 Agent files

Support user/project/plugin agent Markdown files with YAML frontmatter and Markdown system prompt body.

Support and validate documented fields including:
- name;
- description;
- tools;
- disallowedTools;
- model;
- permissionMode;
- maxTurns;
- skills;
- mcpServers;
- hooks;
- memory;
- background;
- effort;
- isolation;
- color;
- additional fields in the current public schema.

Unknown fields are preserved and reported if they affect semantics byv-agent does not implement.

### 7.2 Runtime mapping

- Agent definition selects tools, model, effort, context budget, memory scope, permission mode, skills, and MCP scope.
- Parent agent passes a bounded task packet and relevant context, not the entire raw transcript by default.
- Results return as structured summary plus artifact references and completion status.
- Worktree isolation is used for parallel writers when configured.
- Subagent tool permissions cannot exceed global policy.
- Memory is explicit and user-controllable.
- Nested delegation has depth, concurrency, and budget limits.

Agent Teams and other peer-to-peer team semantics are considered separate advanced compatibility work. Do not label them supported until task coordination, messaging, shutdown, and lifecycle behaviors are implemented and tested.

## 8. Hooks

### 8.1 Hook configuration

Support documented hooks configuration in settings and plugin hook files. Each hook group includes event, optional matcher, and handler list. Preserve handler-specific fields.

### 8.2 Handler types

Target supported public handler types:
- command;
- HTTP;
- prompt;
- agent;
- MCP tool.

Implement only with the required input/output contract, permission mediation, timeout, and diagnostics.

### 8.3 Lifecycle event registry

Use a versioned event registry derived from the current upstream reference. Initial coverage must include the main agent/session lifecycle, tool lifecycle, permission events, compaction, and subagent lifecycle. The registry must be extensible so newly documented events can be added without rewriting the dispatcher.

At minimum evaluate support for:
- SessionStart
- SessionEnd
- UserPromptSubmit
- PreToolUse
- PostToolUse
- PostToolUseFailure
- PermissionRequest
- Notification
- Stop
- SubagentStart
- SubagentStop
- PreCompact
- PostCompact
- FileChanged
- CwdChanged
- WorktreeCreate
- WorktreeRemove
- ConfigChange
- TaskCompleted
- TeammateIdle

The exact event list and fields must be pinned to the compatibility documentation version; upstream can add, rename, or change events.

### 8.4 Hook semantics

- Matchers must use documented matching semantics.
- Command hooks receive documented JSON on stdin and are executed through a restricted runner.
- HTTP hooks use explicit timeout and response validation.
- Prompt/agent hooks use a dedicated model invocation and budget.
- MCP tool hooks call only a selected trusted server and declared tool.
- Exit codes and output fields have event-specific semantics.
- Blocking/allow/deny decisions are applied only when valid for that event.
- Hook failures are logged and surfaced; failure policy (continue, block, or fail task) is explicit per event.
- Hooks cannot bypass permission policy or modify protected configuration.

## 9. Settings compatibility

Support documented:
- user settings;
- shared project settings;
- project-local settings;
- managed settings where an administrator explicitly provides them;
- CLI/session overrides.

Implement exact, documented precedence for the supported baseline. Arrays and object merges must follow upstream semantics per setting rather than a generic deep merge.

Security restrictions must not be weakened by a less-trusted setting source. Keep byv-agent-specific settings in a separate namespace or file rather than polluting Claude-owned settings.

## 10. Compatibility test suites

Maintain fixture directories for:
- Skill frontmatter valid/invalid/unknown fields.
- Skill invocation and arguments.
- Skill resource references.
- Plugin manifests and marketplace catalogs.
- Agent frontmatter and tool restrictions.
- MCP stdio/HTTP/SSE/WebSocket configuration.
- Hook input/output and event matching.
- Settings precedence.
- Duplicate names and scope collisions.
- Unsupported components and warnings.

Every fixture must record upstream documentation URL and review date. Release notes must state the tested compatibility baseline.

## 11. Official references

- Skills: https://code.claude.com/docs/en/skills
- Plugins: https://code.claude.com/docs/en/plugins
- Hooks: https://code.claude.com/docs/en/hooks
- Subagents: https://code.claude.com/docs/en/sub-agents
- MCP: https://code.claude.com/docs/en/mcp
- Settings: https://code.claude.com/docs/en/settings
