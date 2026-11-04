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
5. Composer/status footer: multiline input, slash completion, context/usage meter, queue count, keyboard hints.

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

Requirements:
- Render Markdown safely.
- Render code blocks with language and optional line numbers.
- Render file diffs with additions/deletions.
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
- tool name and concise argument summary;
- command and working directory;
- exit code and duration;
- changed file paths;
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
- Queueing and interrupting are separate actions.

## 12. Keyboard interactions

Initial keymap:
- Enter: submit.
- Shift+Enter: newline.
- Ctrl+C: interrupt active generation/process; if idle, follow documented quit behavior.
- Esc: close popup/modal; do not silently discard composer content.
- Ctrl+L: clear visible transcript viewport, not session history.
- Ctrl+T: toggle Task panel.
- Ctrl+O: toggle Activity panel.
- Ctrl+P: model/provider picker.
- Ctrl+E: effort picker.
- Ctrl+G: task/goal view.
- Tab/Shift+Tab: move focus.
- /: slash-command completion when composer starts empty.
- ? or F1: help/keymap.

Keybindings are configurable and platform-aware. Test Windows Terminal, PowerShell, common macOS terminals, and common Linux terminals.

## 13. Status footer

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

## 14. Approval UI

Display exact tool/action, command or method, working directory, target paths/hosts, risk, reason, grant scope/duration, and actions:
- Allow once;
- Allow for narrow session scope;
- Deny;
- Inspect details.

Never preselect broad or permanent approval.

## 15. Context and compaction UI

Show context usage meter, exact/estimated source label, context breakdown, auto-compact state, threshold/target, compaction start/end, old/new estimate, and recovery warning.

Never present local token estimation as provider-exact. Compaction must be visible without flooding the transcript.

## 16. Subagents and multi-task UX

- Group child tasks under parent task.
- Show subagent roles and states.
- Show delegation and result return.
- Show isolated worktree for writing agents.
- Warn about workspace conflicts.
- Allow inspect/pause/cancel when supported.
- Parent remains responsible for integration and verification.

## 17. Overlays and panels

- Model/provider selector.
- Effort selector.
- Permission approval.
- Plan approval.
- Slash command picker.
- Skills/Agents browser.
- MCP server status.
- Plugin manager.
- Session picker.
- Checkpoint/rewind picker.
- Task details.
- Activity details.
- Help/keymap.
- Error details.

Use one consistent overlay and focus model rather than separate terminal apps.

## 18. Accessibility and terminal compatibility

- Color always has text/icon equivalent.
- No-color mode.
- Reduced-motion mode.
- ASCII fallback when Unicode box drawing is unsupported.
- Handle resize, focus, paste, selection, scrollback.
- Restore terminal on normal exit, Ctrl+C, exception, and provider failure.
- Provide plain CLI fallback when TUI is unsupported.

## 19. Performance and telemetry

- Render from typed events.
- Batch high-frequency token deltas.
- Do not rerender the entire task tree for every token.
- Bound visible transcript/activity while preserving history in storage.
- Keep indexing, diff generation, and syntax analysis off the rendering path.
- Add render diagnostics.
- Do not log secrets or hidden reasoning.

## 20. Acceptance criteria

- Responsive wide/medium/narrow/short layout.
- Conversation, task, activity, and composer state stay consistent.
- Thinking indicator uses public states only.
- Tool cards show actual start/result/error.
- Task progress comes from Task Manager events.
- Completion is distinct from verification.
- Queue and interrupt are distinct.
- Approval shows exact operation and risk.
- Context/compaction reflects measured/estimated values honestly.
- Diff shows the correct file revision.
- Keybindings are tested and remappable.
- Terminal restores on every shutdown path.
- OpenTUI native assets load in source and standalone modes on every advertised target.
- Plain CLI fallback works.
- No API keys, secrets, or hidden reasoning appear in activity logs.

## 21. References

- https://opentui.com/docs/
- https://opentui.com/docs/getting-started/runtime-support/
- https://opentui.com/docs/core-concepts/renderer/
- https://opentui.com/docs/bindings/react/
- https://opentui.com/docs/components/embedded-terminal/
