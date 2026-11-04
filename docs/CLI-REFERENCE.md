# byv-agent CLI and Interactive UX Specification

## 1. Runtime and invocation

Source development uses Bun. The exact entrypoint is defined when the CLI package is implemented.

Standalone release invocation:
- Unix-like: `./byv --help`
- Windows: `byv.exe --help`

A published standalone artifact embeds the Bun runtime. Users must not need Node.js or Bun installed to use it. External executables such as Git, Docker, language servers, and browser engines remain optional system dependencies diagnosed by `byv doctor`.

## 2. Interface principles

- CLI is the first-class interface.
- Text generation, tool activity, permission prompts, and background status must be distinguishable.
- Streaming output must not corrupt terminal rendering.
- Keyboard interrupt must stop or request a safe cancellation.
- All interactive commands must have documented non-interactive equivalents where practical.
- Do not require a TUI for scripted usage.

## 2. CLI command tree

### Main commands

- byv: start interactive session in current workspace.
- byv init: initialize byv-agent project configuration and optional instruction templates.
- byv run: execute a non-interactive task.
- byv resume: resume a session by ID or selector.
- byv sessions: list, inspect, rename, export, import, archive, or delete sessions.
- byv providers: list, add, edit, remove, test, and inspect providers.
- byv models: list, inspect, configure aliases, and select defaults.
- byv skills: list, inspect, validate, and reload Skills.
- byv mcp: list, add, remove, enable, disable, inspect, authenticate, and test servers.
- byv plugin: list, marketplace, install, update, enable, disable, remove, validate, and inspect.
- byv agents: list, inspect, validate, and reload agent definitions.
- byv config: inspect and change configuration.
- byv doctor: diagnose runtime, provider, extension, permission, and storage issues.
- byv version: print version and compatibility baseline.

### Common flags

- --model
- --provider
- --effort
- --mode
- --permission-mode
- --workspace
- --session
- --continue
- --resume
- --non-interactive
- --output-format
- --json
- --verbose
- --debug
- --no-color
- --dry-run
- --settings
- --add-dir
- --sandbox

Flags must be validated against the current command. Do not accept a flag and silently ignore it.

## 3. Interactive slash command catalog

### Session and mode
- /help
- /clear
- /exit
- /quit
- /plan
- /normal
- /auto
- /ask
- /debug
- /review
- /resume
- /sessions
- /branch
- /export

### Provider and reasoning
- /provider
- /model
- /effort
- /usage
- /cost
- /stats

### Context
- /context
- /compact
- /autocompact
- /memory

### Extensions
- /skills
- /agents
- /mcp
- /plugin
- /hooks
- /reload-plugins where compatible

### Safety and execution
- /permissions
- /approve
- /deny
- /shell
- /background
- /queue
- /tasks
- /goal
- /stop

### Change management
- /diff
- /checkpoints
- /undo
- /rewind
- /restore

### Diagnostics
- /config
- /status
- /doctor
- /debug-log
- /schedule
- /jobs

The slash catalog is a byv-agent UX target. Aliases that mimic Claude Code names must be behavior-tested and must not imply support for behavior that has not been implemented.

## 4. Effort UX

- /effort shows current, requested, and effective levels.
- /effort auto|low|medium|high|xhigh|max sets session default.
- Per-turn override should be possible through an explicit command or flag.
- UI must display whether the level mapped natively, was downgraded, or is unsupported.
- A model switch recalculates the effective capability.

## 5. Context and compaction UX

- /context shows model context limit, estimate, provider-reported usage when available, output reserve, and estimated remaining capacity.
- /context breakdown shows context segments and provenance.
- /autocompact shows or updates threshold, target, and enabled state.
- /compact starts manual compaction and shows a preview when configured.
- Compaction must never erase session history.
- Status line shows a compact context meter and compaction state.

## 6. Queue and background UX

- New messages can be queued while a task runs.
- An explicit interrupt/steer action is distinct from queueing.
- Queue view shows order, prompt preview, timestamp, and state.
- Users can reorder, remove, or clear pending messages.
- Background view shows job ID, agent, command/task, runner, logs, and status.
- Cancellation must be propagated and confirmed.

## 7. Permission UX

A permission prompt displays:
- exact operation;
- target path/host/command/tool;
- reason and risk category;
- scope and duration of proposed grant;
- allow once, allow for scoped session, or deny choices.

Never preselect broad or permanent permission grants.

## 8. Status line

Suggested fields:
- project/workspace;
- current model and provider;
- effort;
- mode and permission mode;
- context usage;
- token/cost usage;
- active task and queue count;
- background jobs;
- session ID shortened.

Allow user customization while protecting secret values.

## 9. Input editor and completion

- Multiline input.
- Prompt history.
- Slash command completion.
- Skills and agent completion.
- File/path completion.
- Argument hints.
- Escaping and multiline paste handling.
- Shell command mode with visible distinction from agent messages.
- Keyboard shortcuts documented and configurable.

## 10. Structured output

Non-interactive mode should support:
- text;
- JSON events;
- JSON final result;
- machine-readable exit codes.

Define stable event schemas and version them. Never emit secrets in machine-readable output.
