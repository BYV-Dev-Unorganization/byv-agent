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

## 3. CLI command tree

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

## 4. Interactive slash command catalog

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
- /btw <question>
- /btw history
- /btw clear
- /btw cancel [id]

### Provider and reasoning
- /provider
- /model
- /effort
- /usage
- /cost
- /stats
- /theme [byv-dark|byv-light|auto]

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

`/btw <question>` is an interactive, single-response side channel. It can run during an active task, uses a frozen snapshot of already-known context, has no tool access, and is excluded from the main transcript/context/queue. Its provider usage and cost are tracked separately. See [Side Questions](SIDE-QUESTIONS.md).

## 5. Effort UX

- /effort shows current, requested, and effective levels.
- /effort auto|low|medium|high|xhigh|max sets session default.
- Per-turn override should be possible through an explicit command or flag.
- UI must display whether the level mapped natively, was downgraded, or is unsupported.
- A model switch recalculates the effective capability.

## 6. Context and compaction UX

- /context shows model context limit, estimate, provider-reported usage when available, output reserve, and estimated remaining capacity.
- /context breakdown shows context segments and provenance.
- /autocompact shows or updates threshold, target, and enabled state.
- /compact starts manual compaction and shows a preview when configured.
- Compaction must never erase session history.
- Status line shows a compact context meter and compaction state.

## 7. Queue and background UX

- New messages can be queued while a task runs.
- An explicit interrupt/steer action is distinct from queueing.
- Queue view shows order, prompt preview, timestamp, and state.
- Users can reorder, remove, or clear pending messages.
- Background view shows job ID, agent, command/task, runner, logs, and status.
- Cancellation must be propagated and confirmed.

## 8. Permission UX

A permission prompt displays:
- exact operation;
- target path/host/command/tool;
- reason and risk category;
- scope and duration of proposed grant;
- allow once, allow for scoped session, or deny choices.

Never preselect broad or permanent permission grants.

## 9. Status line

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

## 10. Input editor and completion

- Multiline input.
- Prompt history.
- Slash command completion.
- Skills and agent completion.
- File/path completion.
- Argument hints.
- Escaping and multiline paste handling.
- Shell command mode with visible distinction from agent messages.
- Keyboard shortcuts documented and configurable.

## 11. Structured output

Non-interactive mode should support:
- text;
- JSON events;
- JSON final result;
- machine-readable exit codes.

Define stable event schemas and version them. Never emit secrets in machine-readable output.


## 12. BITE Terminal UI

The full screen design is documented in [TUI-UX-SPECIFICATION.md](TUI-UX-SPECIFICATION.md).

Primary panels:
- Conversation;
- Side-question overlay (separate from the main transcript);
- Task;
- Activity;
- Files;
- Agents;
- Jobs.

Keymap highlights:
- Esc: dismiss the topmost popup/overlay; during an active turn request cancellation; never silently discard the draft or exit.
- Ctrl+C: interrupt foreground work; when idle, open quit confirmation.
- Shift+Tab: cycle interaction mode (Normal, Plan, Auto, Ask, Debug, Review); does not change permission grants.
- Tab: accept completion or move focus forward; Ctrl+Tab / Ctrl+Shift+Tab move between panel focus groups.
- Ctrl+K: command palette; Ctrl+F: search current view.
- Ctrl+T: Tasks; Ctrl+O: Activity; Ctrl+G: goal/task view.
- Ctrl+P: provider/model; Ctrl+E: effort; Ctrl+Shift+P: permission-mode picker.
- ?: help; /keymap: show effective bindings.

`/theme` previews or changes the TUI theme without interrupting active work. Built-in presets and the semantic palette are documented in [TUI-UX-SPECIFICATION.md](TUI-UX-SPECIFICATION.md). Tool cards show action-specific badges and verified line statistics where available. The complete context-aware keymap, including approval, composer, panel, queue, file, job, and terminal fallback behavior, is documented in [TUI-UX-SPECIFICATION.md](TUI-UX-SPECIFICATION.md). Keybindings must be configurable and tested against terminal/platform conflicts. Shift+Enter is used for newline only where the terminal sends an unambiguous sequence; Ctrl+J is the documented fallback.
