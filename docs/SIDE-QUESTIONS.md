# Side Questions — /btw

## 1. Purpose

Side Questions let a user ask a short, context-aware question while the main agent turn continues. The main task must continue without pausing, restarting, being steered, or having its queue changed.

The command is inspired by the user-facing behavior of Claude Code's /btw, but this document defines byv-agent's own runtime and UI contract.

> Ask about what byv-agent already knows, while it keeps doing the current task.

## 2. User experience

- Invoke /btw <question> from the interactive composer at any time, including while the main agent is generating, executing a tool, waiting for a process, or coordinating subagents.
- The command is dispatched immediately to the Side Question Service. It is never appended to the main message queue.
- The main task continues with no state transition caused by the side question.
- A separate overlay/panel shows the side question and streams its answer asynchronously.
- The user can dismiss the overlay and return to the main task at any time. Dismissing the overlay does not cancel the side request or the main task.
- Multiple side questions may be submitted; they are separate requests with their own IDs and bounded concurrency.
- Each side question produces one answer. To ask another question, submit another /btw command.

Examples:

    /btw What does the retry policy do?
    /btw Why did we choose SQLite for session storage?
    /btw What does the current error message mean based on the logs already shown?

## 3. Context snapshot and isolation

At invocation, capture an immutable, versioned snapshot of the main session's available context.

The snapshot may include:
- completed user and assistant-visible transcript messages;
- completed tool results and artifacts already admitted to the active context;
- the current task goal, accepted plan, decisions, and completed task evidence;
- project instructions and relevant context already loaded for the main session, subject to normal redaction and trust labels.

The snapshot must exclude:
- the assistant response currently being generated;
- incomplete tool calls and partial/uncommitted tool output;
- hidden chain-of-thought, private reasoning tokens, internal scratchpads, or confidential deliberation;
- API keys, authorization headers, and secret values;
- tool schemas and executable capabilities.

The side question can answer only from this supplied snapshot. It must clearly say when the answer requires new investigation or information that is not present. It must not pretend to have inspected a file, run a command, queried an MCP server, browsed the web, or verified a fact when it has not.

Snapshot identity and transcript revision are recorded with the side-question request so answers remain explainable even while the main task advances.

## 4. No-tool execution boundary

A side question is read-only and has no tools. The provider request must not include tool definitions or tool routing capability.

Forbidden actions include:
- reading or writing files;
- running shell commands, tests, builds, Git operations, or scripts;
- invoking MCP tools, Plugins, Hooks, Skills that trigger tools, Agents, subagents, or browser automation;
- making web/network requests through an agent tool;
- changing task goals, plans, task state, permissions, configuration, memory, or the message queue.

If the model emits a tool call or asks to perform an action, the runtime must reject it without dispatch. Render a clear note that /btw is read-only and suggest a normal task or a deliberate fork when investigation/action is needed.

Permission grants for the main session do not grant side-question tool access.

## 5. Main-task non-interference

The side-question service has its own request ID, abort controller, provider stream, timeout, retry policy, usage record, and concurrency slot.

It must not:
- acquire the main agent-loop lock;
- stop, pause, restart, or steer the main turn;
- change task, subagent, background-job, approval, or queue state;
- alter the main transcript or active context pointer;
- trigger compaction or consume the main task's context budget;
- run user hooks or mutate project/session configuration.

Side-question provider failures, timeouts, cancellations, malformed responses, and rate limits are isolated. They must never fail or cancel the main task.

The main provider may be used concurrently if it supports concurrent requests. A separate model profile may be configured for side questions. Provider rate limits still apply; usage and cost are tracked as a separate category and are not zero-cost.

## 6. Transcript, context, and persistence

- Side-question prompt and answer are stored in a separate side-question record/table, not as ordinary transcript turns.
- They are excluded from normal prompt assembly, task history, memory extraction, compaction input, transcript export, and queue processing by default.
- Side-question usage is reported separately in session statistics and cost reporting.
- Keep a bounded recent-history list in the overlay (default five previous exchanges); older records may remain available according to retention configuration.
- /btw clear clears the visible side-question history for the current session; it must explain whether records are merely hidden or permanently deleted. The default operation is non-destructive hiding.
- Side-question records are covered by session retention and deletion policy, with the same secret redaction and local privacy protections as other session data.

## 7. Forking and promotion

An optional f action on a completed side answer may fork a new session from the captured snapshot and promote that side question and answer into real transcript turns. The new session has normal tool access subject to its own permission policy. The original session and active task remain unchanged.

Forking is an explicit user action. A side answer is never automatically promoted into the main transcript or task goal.

## 8. Commands

- /btw <question>: start a single-turn side question.
- /btw history: show recent side questions for this session.
- /btw clear: clear the visible history list without deleting durable records.
- /btw cancel [id]: cancel a pending side-question request only; never cancel the main task.
- Empty /btw: show usage and examples.

The feature is available in interactive CLI/TUI sessions. Non-interactive byv run must reject it with a clear explanation rather than silently treating it as a normal prompt.

## 9. Overlay keyboard behavior

When the side-question overlay is focused:
- Space, Enter, or Esc: dismiss the overlay and return to the previous focus; an in-flight side request continues.
- Up / Down: scroll the current answer.
- c: copy the answer as raw Markdown.
- f: fork the captured context plus the question/answer into a new session (completed answers only).
- x: clear earlier exchanges from the visible overlay list without deleting durable records.

The overlay must not steal focus from, or overwrite, the main composer draft. Keyboard handling is local to the overlay while it is focused.

## 10. Events and storage model

Suggested typed events:
- side_question.requested
- side_question.started
- side_question.delta
- side_question.completed
- side_question.failed
- side_question.cancelled
- side_question.dismissed
- side_question.forked

Each record should include:
- side_question_id;
- parent session ID and captured transcript/context revision;
- question and answer;
- status and timestamps;
- provider/model profile and request correlation ID;
- input/output token usage and cost when available;
- error classification and cancellation reason;
- whether it was forked and the resulting session ID, if any.

A dedicated side_questions table/repository is preferred. It must never be joined into the ordinary transcript query used for prompt assembly.

## 11. Configuration

Suggested settings:

    {
      "sideQuestions": {
        "enabled": true,
        "modelProfile": null,
        "maxConcurrent": 2,
        "maxOutputTokens": 1024,
        "timeoutMs": 60000,
        "visibleHistoryLimit": 5,
        "retention": "session"
      }
    }

These are initial design defaults, not provider guarantees. Validate all values. A configured side-question model must be a valid provider/model profile. If it is unavailable, fail the side question clearly; do not fall back to the main task queue.

## 12. Failure and edge-case behavior

- If no stable context snapshot can be captured, reject the side question without affecting the main task.
- If a side question arrives during an incomplete tool protocol exchange, snapshot only the last committed state.
- If max concurrency is reached, reject with a retry hint or keep a clearly labelled side-question-only pending slot; never put it into the main message queue.
- If the provider rate-limits the side request, show the retry state separately and do not stall the main request.
- If the side answer is incomplete due to cancellation/timeout, label it partial.
- On process restart, restore durable completed side-question history. Pending requests may be marked interrupted unless the runtime can safely resume them.
- A side question submitted while idle still uses the same isolated path and remains outside the main transcript.
- If a side question asks for new investigation or action, explain the limitation and offer the user a normal task or explicit fork.

## 13. Acceptance summary

The feature is complete only when:
- /btw works while the main turn is actively generating and while tools/processes are running;
- the main task's state, output, queue, cancellation token, and context pointer remain unchanged;
- side requests run asynchronously and return without waiting for the main turn to finish;
- the request contains a frozen, redacted context snapshot and no tools;
- attempted tool calls are rejected and never dispatched;
- side prompts/answers never enter main transcript, context, compaction, memory, or task queue by default;
- usage and cost are reported separately;
- overlay dismissal never cancels the main task or clears the main draft;
- concurrency, cancellation, timeout, provider-limit, restart, and fork behavior are tested.