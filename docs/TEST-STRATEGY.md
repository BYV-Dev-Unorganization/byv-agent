# byv-agent Test and Evaluation Strategy

## 1. Test philosophy

byv-agent combines nondeterministic model output with deterministic local execution. Test deterministic infrastructure independently from model quality, and evaluate end-to-end coding performance with repeatable task fixtures.

No release claim should be based only on a successful manual demo.

## 2. Test layers

### Unit tests
- Config parsing and precedence.
- Provider request construction and stream parsing.
- Canonical message/event transformations.
- Effort resolution and mapping.
- Context budget calculation.
- Compaction selection and validation.
- Tool schema validation.
- Permission evaluation.
- Path normalization.
- Git/checkpoint state.
- Skill/agent/plugin frontmatter parsing.
- MCP config parsing.
- Hook matcher and lifecycle dispatch.
- Session transitions and queue ordering.

### Contract tests
- OpenAI-compatible adapter.
- Anthropic Messages adapter.
- Optional OpenAI Responses adapter.
- MCP client and transports.
- Runner implementations.
- Plugin/marketplace source adapters.
- Storage repositories.

### Integration tests
- Fake provider + real tool registry.
- Fake tool runner + agent loop.
- SQLite persistence and migrations.
- Compaction and session resume.
- Skill invocation with tool permissions.
- MCP server tool call through the provider loop.
- Hook execution around tool lifecycle.
- Plugin install/load/remove.
- Subagent delegation and result merge.
- Background task recovery.

### End-to-end tests
- Start CLI in a fixture repository.
- Ask the agent to inspect, edit, test, and report.
- Verify filesystem diff and test result.
- Exercise permission prompts.
- Cancel running command.
- Resume session after process restart.
- Trigger context compaction.
- Run parallel agents in isolated worktrees.
- Run a remote/container task where available.

### Security tests
- Path traversal and symlink escape.
- Shell injection and unsafe quoting.
- Secret redaction.
- Prompt injection in repository/browser/MCP data.
- Plugin path traversal and malicious install scripts.
- Hook timeout and command denial.
- Tool result size abuse.
- Resource exhaustion.
- Unauthorized network and filesystem access.
- Cross-session data leakage.
- Remote credential scope and cleanup.

## 3. Provider fixture matrix

Each provider fixture should cover:
- valid text response;
- streamed text;
- tool call and tool result;
- multiple calls;
- interleaved text and tool blocks;
- partial/truncated stream;
- usage and cache metadata;
- effort/thinking mapping;
- unsupported parameter behavior;
- context limit error;
- authentication error;
- rate limit with Retry-After;
- server unavailable;
- timeout;
- cancellation;
- malformed response.

Fixtures should not require real paid API calls. A small opt-in smoke suite may use real providers with an explicit user-provided key and budget.

## 4. Extension compatibility fixtures

Keep versioned fixtures for:
- Skills YAML frontmatter and argument expansion.
- Skill scope and collision rules.
- CLAUDE.md and rules loading.
- Agent frontmatter and tool restrictions.
- Plugin manifest and component layout.
- Marketplace metadata and Git source.
- MCP stdio and remote transport configuration.
- MCP tools/resources/prompts.
- Hook events, matchers, stdin, stdout, exit codes, and decisions.
- Settings scopes and precedence.

Every fixture must identify the upstream documentation page and the compatibility baseline it represents.

## 5. Compaction evaluation

Use long synthetic and real-world transcripts with:
- active task goal;
- multiple pending steps;
- important constraints;
- decisions;
- several tool call/result exchanges;
- large terminal outputs;
- modified file list;
- test failures and results;
- user corrections;
- Skills and MCP context;
- nested subagent summary.

Assertions:
- required facts survive;
- tool calls are never orphaned;
- no action executes twice;
- full transcript is intact;
- active context is valid for the provider;
- failed compaction leaves the old context active;
- token reduction is measured;
- summary provenance points to source messages/artifacts.

Use both deterministic structural checks and a rubric-based model evaluation. Do not rely only on an LLM judge.

## 6. Coding task evaluation suite

Maintain a fixed benchmark repository set with representative tasks:
- understand an unfamiliar repository;
- add a small feature;
- fix a reproducible bug;
- refactor without changing behavior;
- add unit tests;
- repair failing tests;
- update docs and examples;
- perform a read-only review;
- resolve a controlled merge conflict;
- make a multi-file change with migrations;
- implement a task that requires MCP;
- use a Skill or plugin workflow.

For each task store:
- starting repository revision;
- user prompt;
- environment and dependency lock;
- expected behavior;
- allowed commands/tools;
- hidden or explicit tests;
- success rubric;
- known failure modes.

## 7. Metrics

Track:
- task success rate;
- test pass rate;
- regression rate;
- unauthorized action count (must be zero);
- tool call count;
- provider request count;
- context compaction count;
- token usage;
- estimated/provider-reported cost;
- time-to-first-token and total elapsed time;
- recovery rate after failure;
- human intervention count;
- compatibility fixture pass rate.

Compare metrics by provider/model/profile and agent version. Avoid ranking models from a single task.

## 8. Test data and privacy

- Use synthetic repositories for default CI.
- Do not upload user workspaces or transcripts to external evaluators without explicit consent.
- Redact credentials and personal data from fixtures.
- Keep production telemetry opt-in.
- Include license/source attribution for external benchmark repositories.

## 9. CI gates

Pull request checks:
- formatting;
- lint;
- type checking;
- unit tests;
- package builds;
- compatibility fixtures;
- security tests relevant to changed subsystem.

Release gates:
- full integration/e2e suite;
- provider adapter contract suite;
- migration and recovery;
- platform matrix;
- coding evaluations;
- security review;
- documentation and release-note audit.
