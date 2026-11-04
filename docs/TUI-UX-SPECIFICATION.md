# BYV Interactive Terminal Experience (BITE)
## Terminal UI/UX Specification

**Product:** byv-agent  
**Organization:** BYV Dev Unorganization  
**Status:** Design specification  
**Runtime:** Bun  
**Renderer:** OpenTUI Core  
**Platforms:** Windows, Linux, macOS. Android is not in scope.

## 1. Product intent

The terminal interface is a first-class coding workspace, not a colored log viewer. At any point, users should understand:
1. What the agent is doing now.
2. What goal it is pursuing.
3. Which task steps are complete, active, blocked, or waiting.
4. Which files, commands, Git operations, MCP tools, and subagents are involved.
5. Whether the agent is waiting for the model, a tool, user approval, or a background process.
6. How much context, tokens, and budget have been used.
7. How to interrupt, steer, approve, inspect, or resume work.

The interface is keyboard-first, responsive to terminal dimensions, low-noise, accessible, and honest about the information the runtime actually has.

## 2. Product identity

- Organization: BYV Dev Unorganization.
- Product: byv-agent.
- Terminal experience name: BITE — BYV Interactive Terminal Experience.
- Visual identity: dark neutral base, BYV blue accent, restrained semantic colors.
- Green = completed/success; cyan/blue = active; amber = waiting/attention; red = error/denied; gray = inactive.
- Never rely on color alone; always show a label or icon.
- Respect no-color and reduced-motion modes.
- Do not imitate another vendor's branding or proprietary UI.

### 2.1 Theme system and semantic palette

The first built-in preset is **BYV Dark**. Theme tokens are semantic: components must request a role such as `tool.edit` or `status.running`, never hard-code terminal colors throughout the UI.

Base palette (design values; renderer maps them to truecolor, 256-color, or the closest supported terminal palette):

| Token | BYV Dark value | Use |
|---|---:|---|
| `surface.base` | `#0B1020` | Main terminal background |
| `surface.panel` | `#111827` | Conversation/tool/task panels |
| `surface.elevated` | `#1F2937` | Popups, selected rows, overlays |
| `text.primary` | `#F3F4F6` | Main content |
| `text.secondary` | `#CBD5E1` | Metadata and descriptions |
| `text.muted` | `#94A3B8` | Timestamps and subdued labels |
| `brand.primary` | `#3B82F6` | BYV brand accent and focus |
| `status.running` | `#38BDF8` | Running/active |
| `status.success` | `#22C55E` | Completed successfully |
| `status.warning` | `#F59E0B` | Waiting, approval, attention |
| `status.error` | `#EF4444` | Failed, denied, destructive |
| `status.inactive` | `#64748B` | Idle, skipped, cancelled |
| `diff.added` | `#22C55E` | Added lines |
| `diff.removed` | `#F87171` | Removed lines |

Tool action accents are a separate color channel from execution status. An `EDIT` badge may retain its action color while its status changes from RUNNING to DONE or ERROR; never use an action color alone to imply success.

| Action badge | Accent token | Examples |
|---|---|---|
| `READ`, `SEARCH`, `GLOB` | `tool.read` = cyan `#22D3EE` | Read file, find text, list files |
| `WRITE`, `CREATE` | `tool.write` = blue `#60A5FA` | Create or write a file |
| `EDIT`, `PATCH`, `UPDATE` | `tool.edit` = violet `#A78BFA` | Replace content or apply a patch |
| `DELETE`, `REMOVE` | `tool.delete` = red `#F87171` | Delete a file/resource |
| `MOVE`, `COPY`, `RENAME` | `tool.move` = magenta `#E879F9` | Move, copy, or rename |
| `SHELL`, `COMMAND`, `PROCESS` | `tool.command` = amber `#FBBF24` | Run a process or command |
| `GIT` | `tool.git` = teal `#2DD4BF` | Git status, diff, branch, commit |
| `MCP`, `PLUGIN`, `HOOK`, `AGENT` | `tool.extension` = purple `#C4B5FD` | Extension or delegated operation |

Theme requirements:
- Keep action category, execution state, and diff state visually independent.
- Use labels/icons as well as color; support no-color and low-color terminals with text labels and patterns.
- Maintain WCAG-informed contrast for foreground text where terminal color capability permits; do not rely on subtle hue differences alone.
- Support `byv-dark`, a light preset, and user-defined semantic token overrides. BYV Dark is the default.
- `auto` color mode may follow terminal capability/environment; never assume a dark terminal background without detection or a configured fallback.
- Theme changes apply immediately and never restart or interrupt an agent task.
- Keep theme tokens centralized so tool cards, diff views, status indicators, approvals, and popups remain consistent.

## 3. Renderer and runtime

Use @opentui/core as the initial renderer. OpenTUI provides terminal rendering, layout, input, scrolling, text input, selection, Markdown, code, diff, tables, embedded terminal, and lifecycle APIs.

Implementation direction:
- Bun is the only application runtime.
- Use OpenTUI Core renderables and event APIs first.
- Create a small BYV component layer for reusable visual elements.
- @opentui/react is optional; adopt it only after it passes the same Bun and standalone executable compatibility matrix and materially improves maintainability.
- Do not add Ink as a second renderer.
- TUI components consume typed session, agent, task, tool, and usage events. They never call model providers or execute tools directly.

### Native compatibility gate

OpenTUI Core uses native artifacts. Its current public documentation lists Bun 1.3.14 or later, and Bun 1.4 or later for native Windows ARM64 FFI. Native artifacts are published for macOS x64/arm64, Linux glibc and musl x64/arm64, and Windows x64/arm64. Availability does not prove complete runtime or compiled-executable parity.

Before declaring a target supported:
- run a Bun/OpenTUI compatibility spike;
- verify native FFI and library loading in source and standalone executable modes;
- verify embedded native assets and Tree-sitter assets if used;
- test each OS, CPU architecture, and Linux libc;
- document the exact working Bun/OpenTUI versions;
- provide a basic line-oriented CLI fallback if the renderer cannot start.

References:
- https://opentui.com/docs/
- https://opentui.com/docs/getting-started/runtime-support/
- https://opentui.com/docs/core-concepts/renderer/
- https://opentui.com/docs/components/embedded-terminal/
- https://opentui.com/docs/bindings/react/

## 4. Screen architecture

Five logical regions:
1. Header: product, workspace, session, provider/model, mode, effort, permission mode.
2. Conversation: user messages, assistant responses, plans, summaries, collapsible tool cards.
3. Task panel: goal, checklist, dependencies, progress, child tasks, subagent assignment.
4. Activity panel: current public activity status, elapsed time, recent tool events, background jobs.
5. Composer/status footer: multiline input, slash completion, context/usage meter, queue count, side-question activity, keyboard hints.

Conversation, task, and activity views are projections of the same event/state source. They must not keep separate competing task states.

## 5. Responsive layout

Breakpoints are guidelines; always calculate using actual terminal rows and columns.

### Wide (about 120+ columns)
- Header across full width.
- Main conversation around 60–70%.
- Right panel around 30–40%, with Task and Activity tabs.
- Optional left session/project rail only when enough room remains.

### Medium (about 90–119 columns)
- Hide left rail by default.
- Keep conversation primary.
- Right panel becomes a toggleable drawer or compact column.

### Narrow (below about 90 columns)
- Single-column layout.
- Conversation takes priority.
- Task and Activity are switchable views.
- Header collapses to product, model, and active state.
- Long tool details wrap or collapse.

### Short terminal (below about 24 rows)
- Compact mode.
- Hide decorative metadata.
- Keep active state, current task, composer, approval, and error visible.
- Other information is accessible using panel toggles.

On resize, preserve composer content, focus, selected task, and scroll position.

## 6. Header

Show:
- byv-agent brand.
- workspace/project basename.
- short session ID.
- provider/model alias.
- mode: Normal, Plan, Auto, Ask, Debug, Review.
- effort level.
- permission mode.

Never show API keys, private headers, or secret references.

Example:

    byv-agent | ~/work/api | session 8F21A
    provider/model | EFFORT HIGH | PLAN | ASK-BEFORE-EXEC

## 7. Conversation view

Message/card types:
- user message;
- assistant response;
- plan proposal;
- public progress update;
- tool call;
- tool result;
- approval request;
- compaction notice;
- subagent result;
- warning/error;
- final task summary.

### 7.1 Structured tool cards

Every tool invocation is rendered as a compact, typed card instead of an unstructured line of prose. The card has separate visual fields for action category and execution status.

Required fields when available:
- action badge: `READ`, `WRITE`, `EDIT`, `UPDATE`, `DELETE`, `MOVE`, `COMMAND`, `GIT`, `MCP`, etc.;
- target: workspace-relative file path, command summary, MCP server/tool, or resource identifier;
- status: `QUEUED`, `RUNNING`, `WAITING APPROVAL`, `DONE`, `ERROR`, `DENIED`, or `CANCELLED`;
- concise operation summary;
- changed-file count and line statistics when the operation changes files;
- duration and exit code for completed process/tool execution;
- expandable output and diff/artifact reference.

Example card layout:

```text
┌ EDIT  src/auth/session.ts ─────────────── DONE ─┐
│ Refresh token validation                      │
│ +18  -4 lines     1 file changed     0.42s    │
└────────────────────────────────────────────────┘

┌ WRITE  src/config/defaults.ts ───────── RUNNING ┐
│ Creating default provider configuration       │
│ +36 lines (pending verification)              │
└────────────────────────────────────────────────┘

┌ DELETE  src/legacy/token.ts ─────────── ERROR ─┐
│ Permission denied                             │
│ No file change was applied                    │
└────────────────────────────────────────────────┘
```

The exact border glyphs may adapt to terminal capability, but the hierarchy and fields must remain clear.

### 7.2 File-change and line-count contract

- `WRITE` / `CREATE`: show added and removed line counts, final line count when known, and file path.
- `EDIT` / `PATCH` / `UPDATE`: show added and removed lines from the actual before/after diff, plus changed-file count.
- `DELETE`: show removed lines when the pre-delete content is available and confirmed.
- `MOVE` / `RENAME`: show source and destination paths; do not report line changes unless content changed.
- Multi-file operations: show an aggregate summary (files changed, total additions/deletions) and expandable per-file rows.
- Diff signs use `diff.added` and `diff.removed` colors; numbers must be formatted consistently, e.g. `+18` and `-4`.
- Calculate counts from the actual file snapshot/diff or structured tool result, never from model-written prose, planned changes, or the size of tool arguments.
- If exact counts cannot be established, display `line stats unavailable` or omit the metrics. Never estimate or invent line counts.
- For an operation that has started but not completed, label counts as provisional/pending until the runtime confirms the result.
- A denied/failed operation must not show a successful line-change count unless a separately confirmed partial change occurred; show that partial state explicitly.
- Preserve the exact diff artifact/revision referenced by the card so opening it cannot show a later unrelated file version.

### 7.3 Tool-card states and grouping

- Running cards use the running status token and a subtle spinner; do not imply completion.
- Successful cards use the success status token and a `DONE` label.
- Approval-required cards use the warning token and show the pending approval boundary.
- Failed/denied cards use the error token and include the actual error or denial reason.
- Cancelled cards use the inactive token and distinguish cancellation from failure.
- Consecutive tool calls may be grouped under one collapsible operation group, but each call retains its own ID, target, result, and state.
- Keep long output collapsed by default; the card summary remains visible in the transcript and Activity panel.

Requirements:
- Render Markdown safely.
- Render code blocks with language and optional line numbers.
- Render file diffs with additions/deletions.
- Render typed tool cards with action badge, target, execution state, changed-file count, verified line additions/deletions, and duration/exit code where available.
- Color tool actions independently from success/error state, following the semantic BYV palette.
- Collapse long tool output by default; allow inspection of full artifact.
- Distinguish assistant-authored text from runtime-generated status.
- Preserve streaming without flicker.
- Preserve access to original transcript after compaction.
- Copy text/code/command/path when terminal capabilities permit.
- Do not dump every internal event as a chat message.

## 8. Thinking and activity indicator

### Meaning

The thinking indicator is a public activity status, not a view into hidden chain-of-thought. Never display private model reasoning, hidden reasoning tokens, internal scratchpads, or confidential deliberation.

Allowed status labels:
- Understanding request…
- Planning task…
- Inspecting project…
- Searching code…
- Reading files…
- Generating changes…
- Running command…
- Waiting for approval…
- Waiting for provider…
- Calling MCP tool…
- Running tests…
- Reviewing diff…
- Compacting context…
- Waiting for background process…
- Verifying completion…

### Canonical activity states

- idle
- receiving_request
- understanding
- planning
- retrieving_context
- waiting_for_model
- generating_response
- executing_tool
- waiting_for_approval
- waiting_for_external_tool
- running_tests
- reviewing_changes
- compacting_context
- running_subagents
- waiting_for_background_job
- verifying_goal
- completed
- failed
- cancelled

The orchestrator owns state. The UI derives the indicator from typed events. A provider's support for reasoning/thinking does not mean hidden reasoning can be displayed.

### Indicator behavior

Show:
- subtle spinner/pulse for active states;
- short status label;
- elapsed time for long operations;
- provider waiting/streaming status where known;
- current tool name while executing;
- waiting/approval badge for blocked actions;
- error details only after a real failure event.

No animation while idle. Respect reduced motion and plain-terminal mode.

Progress text must be short, factual, generated from actual operation state, and updated only on meaningful changes. Never claim a command or test has started/completed before the runtime confirms it.

## 9. Task panel

The task panel is a first-class feature.

### Task schema

Each task stores:
- stable task ID;
- title and concise goal;
- parent task ID;
- status and priority;
- ordered steps/subtasks;
- dependencies;
- assigned agent/subagent;
- model and effort profile;
- timestamps;
- progress events;
- completion criteria;
- evidence/artifact references;
- blocker or approval reference.

### Task statuses

- queued;
- pending;
- in_progress;
- waiting_approval;
- blocked;
- verifying;
- completed;
- failed;
- cancelled.

Status is owned by Task Manager, not inferred only from assistant prose.

### Example task view

    CURRENT TASK
    Add refresh token support                  IN PROGRESS

    [x] Inspect authentication flow
    [x] Map token and session data model
    [>] Implement refresh token endpoint
    [ ] Add unit and integration tests
    [ ] Run tests and security checks

    2/5 steps complete | 1 active | 0 blocked

Support nested tasks, expand/collapse, dependencies, blockers, task filtering, selection, agent assignment, completion evidence, retries/reopen where allowed, and task history.

### Completion integrity

Distinguish:
- agent claims completion;
- runtime verified completion criteria;
- tests passed;
- tests not run;
- task blocked or partial.

Never show green Completed solely because the model said it is done. Verification evidence is separate.

## 10. Activity panel

Tabs:
- Activity;
- Tasks;
- Files;
- Agents;
- Jobs (when background jobs exist).

Activity entries:
- operation start/completion;
- tool name/action badge and concise argument summary;
- command and working directory;
- exit code and duration;
- changed file paths, changed-file count, verified lines added/removed, and per-file diff stats when available;
- test summary;
- MCP server/tool call;
- approval request/result;
- context compaction;
- provider fallback/retry;
- subagent start/result;
- background job state.

Long output is stored as an artifact and opened on demand. Redact secrets before rendering.

Files view:
- changed/created/deleted files;
- staged/unstaged state where available;
- per-file diff;
- restore/checkpoint action subject to permission.

Agents view:
- agent role/name;
- assigned task;
- model and effort;
- current state;
- token/cost summary;
- isolated worktree;
- result.

Jobs view:
- job ID;
- shell/process/task description;
- runner;
- state;
- elapsed time;
- log tail;
- stop/cancel action.

## 11. Composer

- Multiline prompt editor.
- Enter submits when not inserting a newline.
- Shift+Enter inserts newline where terminal supports it.
- Large paste must not submit unexpectedly.
- Up/Down navigate prompt history only when cursor conditions are safe.
- Tab completes slash commands, Skills, files, agents, and supported mentions.
- Slash suggestions appear in a bounded popup.
- @path completion can refer to project files.
- Preserve composer content across resize and panel navigation.
- Show queued message count separately from current turn state.
- Queueing, interrupting, and side-question dispatch are separate actions.
- While a main turn is active, an explicit `/btw <question>` submitted in the composer is sent to the Side Question Service immediately; ordinary prompts retain the normal queue behavior.

## 12. Side-question overlay

The `/btw` side-question overlay is a separate surface from the main conversation and task panel. It may open while the main agent is generating, executing a tool, waiting on a process, or coordinating subagents.

- Render the question, captured context revision, provider/model, streaming answer, elapsed time, status, and separately reported token/cost usage where available.
- The overlay streams independently; it must not block the main event loop or acquire the main turn lock.
- Dismissing the overlay leaves the side request running and leaves the main task untouched.
- Preserve the main composer draft and focus state when opening/closing the overlay.
- Keep a dimmed list of the five most recent side-question exchanges by default; older items remain subject to retention policy.
- A completed answer may be copied as raw Markdown or explicitly forked into a new session.
- Mark unavailable information clearly. The overlay must not imply it has inspected new files or run tools.
- Side-question cards may appear in the Activity view under their own filter, but never as ordinary main-conversation turns.

Overlay keys while focused: `Space`, `Enter`, or `Esc` dismiss; `Up` / `Down` scroll; `c` copies raw Markdown; `f` forks the captured context plus question/answer into a new session; `x` clears earlier exchanges from the visible overlay list. Dismissal does not cancel the side request. Use `/btw cancel [id]` for explicit side-request cancellation.

The runtime contract is defined in [SIDE-QUESTIONS.md](SIDE-QUESTIONS.md).

## 13. Keyboard interactions and complete keymap

The keymap is context-aware. A focused modal or completion popup receives navigation keys before the underlying panel. The active bindings must be shown in Help and in `/keymap`. Users can remap bindings, and the UI must detect/report terminal combinations that cannot be distinguished reliably.

### 12.1 Global shortcuts

| Shortcut | Action | Behavior and safety |
|---|---|---|
| `Esc` | Contextual dismiss / cancel | Apply the priority order below. Never silently erase the composer draft or exit the application. |
| `Ctrl+C` | Interrupt / quit flow | During a foreground turn or process, request cancellation and show the confirmed cancellation state. When idle, open the quit confirmation; a second `Ctrl+C` while that confirmation is focused exits. |
| `Shift+Tab` | Cycle interaction mode | Cycle `Normal → Plan → Auto → Ask → Debug → Review → Normal`. Update the header and show the resulting mode. This does not grant permissions or weaken policy. |
| `Ctrl+Tab` | Next focus group | Move focus forward between the composer and visible panel groups. |
| `Ctrl+Shift+Tab` | Previous focus group | Move focus backward between the composer and visible panel groups. |
| `Ctrl+K` | Command palette | Open the searchable command/action palette. |
| `Ctrl+F` | Search current view | Search the visible transcript, activity, task, or file list; scope is shown in the search UI. |
| `Ctrl+L` | Clear viewport | Clear visible transcript rendering only. Preserve the session transcript, task state, and artifacts. |
| `Ctrl+T` | Toggle Tasks panel | Open/focus the Tasks view; repeat to return to the previous view. |
| `Ctrl+O` | Toggle Activity panel | Open/focus the Activity view; repeat to return to the previous view. |
| `Ctrl+G` | Goal/task view | Open the current goal and task tree. |
| `Ctrl+P` | Provider/model picker | Open provider and model selection. |
| `Ctrl+E` | Effort picker | Open effort selection and show requested/effective mapping. |
| `Ctrl+Shift+P` | Permission-mode picker | Open the permission-mode selector. This changes a session preference only; runtime policy and managed restrictions remain authoritative. |
| `?` or `F1` | Help/keymap | Open contextual help with the effective keybindings for the current terminal. |

### 12.2 Exact Esc priority

When `Esc` is pressed, handle only the first applicable item:

1. Close the topmost autocomplete, slash-command, command-palette, or side-question overlay and return focus to its owner. Dismissing a side-question overlay does not cancel its request or the main task.
2. Close the topmost non-approval modal or detail overlay and restore the previous focus.
3. In an approval dialog, dismiss the dialog while leaving the operation blocked and unresolved. Dismissing is not approval or denial.
4. During model generation or a foreground agent turn, request cancellation. Preserve streamed output, the draft, queued messages, and task evidence; show cancellation only after the runtime confirms it.
5. In a panel, clear the current selection or collapse the focused detail view.
6. Otherwise, do nothing.

An explicit `/exit` or the idle `Ctrl+C` quit-confirmation flow is required to exit. `Esc` must never be overloaded as an unconditional application quit key.

### 12.3 Composer and completion

| Shortcut | Action |
|---|---|
| `Enter` | Submit the current message when no completion popup or confirmation control is focused. |
| `Shift+Enter` | Insert a newline when the terminal reports this combination distinctly. |
| `Ctrl+J` | Insert a newline as a fallback on terminals that do not distinguish `Shift+Enter`. |
| `Tab` | Accept the highlighted completion when a completion popup is open; otherwise move focus to the next focusable region. |
| `Up` / `Down` | Move the caret within the draft. At the first/last visual line, navigate prompt history when no popup is open. |
| `Ctrl+Up` / `Ctrl+Down` | Navigate prompt history without moving the caret, when supported by the terminal. |
| `Ctrl+Backspace` | Delete the previous word. |
| `Ctrl+A` | Select all text in the composer; it must not select the entire transcript. |
| `Ctrl+Enter` | Submit the draft as an alternate send binding when configured; never treat it as newline. |
| `/` | Open slash-command suggestions only when the composer is empty. Otherwise insert a literal slash. |
| `@` | Open path/agent/Skill mention suggestions when the input context supports them; otherwise insert the literal character. |

Prompt history must not replace a non-empty draft without an explicit user action. Large paste operations must never trigger submission.

### 12.4 Panel and list navigation

| Shortcut | Action |
|---|---|
| `Tab` / `Shift+Tab` | Move to the next/previous focusable control when focus is inside a panel or modal. In the composer, `Tab` follows the completion rule above; global `Shift+Tab` cycles interaction mode unless a modal explicitly documents a local reverse-navigation binding. |
| `Ctrl+Tab` / `Ctrl+Shift+Tab` | Move focus between panel groups, forward/backward. |
| `Up` / `Down` | Move the selected row in a list or task tree. |
| `Left` / `Right` | Collapse/expand a tree node, or move between columns/tabs where applicable. |
| `Enter` | Open the selected item or activate the focused control. |
| `Space` | Toggle the selected checkbox or task-tree expansion where applicable. |
| `PageUp` / `PageDown` | Scroll the focused panel by one page. |
| `Home` / `End` | Move to the start/end of the focused list or panel. |
| `Ctrl+Home` / `Ctrl+End` | Jump to the beginning/end of the conversation transcript. |
| `Backspace` | Return from an item's detail view to its parent list when that view documents this behavior. |

### 12.5 Modal, approval, and selection controls

- `Up` / `Down`: move between modal options.
- `Enter`: activate the focused option.
- `Tab` / `Shift+Tab`: move focus forward/backward within a modal; modal-local behavior takes precedence over global focus navigation.
- `Space`: toggle a focused checkbox.
- `Esc`: follow the exact priority and approval behavior in §12.2.
- In an approval dialog only: `A` = Allow once, `S` = Allow for the displayed narrow session scope, `D` = Deny, `I` = Inspect details. The selected action and scope must be visible before activation. No allow action is preselected.
- Dismissing an approval dialog with `Esc` leaves the operation blocked and pending; it never approves it.

### 12.6 Tasks, files, jobs, and queue

- Tasks: `Enter` opens task details; `Space` expands/collapses the selected task; `Esc` returns to the task list; destructive or state-changing actions still use the permission/confirmation flow.
- Files: `Enter` opens the selected file diff; `Esc` returns to the file list. Restore/revert actions require an explicit confirmation and the normal permission gate.
- Jobs: `Enter` opens job details/logs; `X` requests cancellation of the selected job and requires confirmation when the job may have side effects.
- Queue: `Enter` inspects the selected message; `Delete` removes it after confirmation; `Ctrl+Up` / `Ctrl+Down` reorders it when queue focus is active. Queue operations never interrupt the active turn.

### 12.7 Mode and permission separation

- `Shift+Tab` cycles the interaction mode in the exact order in §12.1 and is equivalent to selecting the matching mode command.
- `/normal`, `/plan`, `/auto`, `/ask`, `/debug`, and `/review` remain available as direct mode commands.
- `Ctrl+Shift+P` and `/permissions` open the permission-mode selector.
- Interaction mode and permission mode are separate state values. Changing interaction mode must never silently change the permission mode, grant a tool, disable approval, expand a filesystem/network scope, or bypass managed policy.
- Any permission-mode change that increases autonomy must be clearly displayed and follow the configured confirmation policy.

### 12.8 Terminal compatibility

Key combinations vary across terminal emulators and operating systems. At startup, detect terminal capabilities where possible and expose the effective keymap in Help. If a combination cannot be distinguished, provide a configurable alternative and a discoverable slash-command equivalent. In particular:
- support `Shift+Enter` only when its sequence is distinguishable; otherwise advertise `Ctrl+J` or the user's configured newline binding;
- do not assume every terminal reports `Ctrl+Shift+P`, function keys, or modified arrow keys identically;
- preserve normal terminal paste, selection, and scrollback behavior;
- test Windows Terminal, PowerShell, common macOS terminals, and common Linux terminals.

Keybindings are configurable and platform-aware. Every default binding must have a conflict test and appear in `/keymap`.

## 14. Status footer

Compact status line fields:
- provider/model;
- effort;
- mode;
- permission mode;
- context usage/model limit;
- input/output token usage when available;
- cost when known;
- current task progress;
- queue count;
- active background jobs;
- compaction state.

Example:

    MODEL coder | EFFORT HIGH | CONTEXT 72% | SESSION $0.18 | TASK 2/5 | QUEUE 1

Distinguish provider-reported usage from local estimates. Unknown cost is not zero.

## 15. Approval UI

Display exact tool/action, command or method, working directory, target paths/hosts, risk, reason, grant scope/duration, and actions:
- Allow once;
- Allow for narrow session scope;
- Deny;
- Inspect details.

Never preselect broad or permanent approval.

## 16. Context and compaction UI

Show context usage meter, exact/estimated source label, context breakdown, auto-compact state, threshold/target, compaction start/end, old/new estimate, and recovery warning.

Never present local token estimation as provider-exact. Compaction must be visible without flooding the transcript.

## 17. Subagents and multi-task UX

- Group child tasks under parent task.
- Show subagent roles and states.
- Show delegation and result return.
- Show isolated worktree for writing agents.
- Warn about workspace conflicts.
- Allow inspect/pause/cancel when supported.
- Parent remains responsible for integration and verification.

## 18. Overlays and panels

- Model/provider selector.
- Effort selector.
- Permission approval.
- Plan approval.
- Slash command picker.
- Skills/Agents browser.
- MCP server status.
- Plugin manager.
- Session picker.
- Side-question answer overlay.
- Checkpoint/rewind picker.
- Task details.
- Activity details.
- Help/keymap.
- Error details.

Use one consistent overlay and focus model rather than separate terminal apps.

## 19. Accessibility and terminal compatibility

- Color always has text/icon equivalent.
- No-color mode.
- Reduced-motion mode.
- ASCII fallback when Unicode box drawing is unsupported.
- Handle resize, focus, paste, selection, scrollback.
- Restore terminal on normal exit, Ctrl+C, exception, and provider failure.
- Provide plain CLI fallback when TUI is unsupported.

## 20. Performance and telemetry

- Render from typed events.
- Batch high-frequency token deltas.
- Do not rerender the entire task tree for every token.
- Bound visible transcript/activity while preserving history in storage.
- Keep indexing, diff generation, and syntax analysis off the rendering path.
- Add render diagnostics.
- Do not log secrets or hidden reasoning.

## 21. Acceptance criteria

- Responsive wide/medium/narrow/short layout.
- Conversation, task, activity, and composer state stay consistent.
- Thinking indicator uses public states only.
- Tool cards show actual start/result/error.
- Task progress comes from Task Manager events.
- Completion is distinct from verification.
- Queue, interrupt, and `/btw` side questions are distinct.
- Side questions stream independently, use no tools, do not enter the main transcript/context, and preserve the active task and composer draft.
- Approval shows exact operation and risk.
- Context/compaction reflects measured/estimated values honestly.
- Diff shows the correct file revision.
- Keybindings are tested and remappable, including contextual `Esc`, global `Shift+Tab` mode cycling, focus navigation, and approval-modal keys.
- `Esc` never silently discards a draft, grants an approval, or exits the application.
- Interaction-mode cycling never changes permission grants or weakens managed policy.
- The effective terminal keymap is discoverable through Help and `/keymap`.
- Terminal restores on every shutdown path.
- OpenTUI native assets load in source and standalone modes on every advertised target.
- Plain CLI fallback works.
- No API keys, secrets, or hidden reasoning appear in activity logs.

## 22. References

- https://opentui.com/docs/
- https://opentui.com/docs/getting-started/runtime-support/
- https://opentui.com/docs/core-concepts/renderer/
- https://opentui.com/docs/bindings/react/
- https://opentui.com/docs/components/embedded-terminal/
