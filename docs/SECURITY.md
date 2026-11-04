# byv-agent Security and Permission Specification

## 1. Security objectives

byv-agent can read source code, edit files, execute commands, connect to external tools, load third-party extensions, and delegate tasks. The security model must protect the user's workspace, machine, credentials, and connected services from accidental actions, malicious repository content, compromised extensions, and prompt injection.

Security is enforced in code, not only through model instructions.

## 2. Threat model

Assume any of the following may be malicious or compromised:
- User prompt content copied from external sources.
- Repository files, README content, comments, tests, and dependency scripts.
- Web pages, issue text, pull request descriptions, and downloaded files.
- MCP server responses and resource content.
- Plugin code, hook scripts, Skills, agents, and marketplace metadata.
- Tool output, terminal logs, and generated artifacts.
- A model response or tool-call argument.

Potential impacts:
- Unauthorized file read/write/delete.
- Secret disclosure.
- Arbitrary command execution.
- Network exfiltration.
- Supply-chain compromise.
- Destructive Git operations.
- Unbounded compute, process, network, or token usage.
- Corruption of session state or task results.

## 3. Trust boundaries

Trust levels:
1. Application-managed code and immutable runtime policy.
2. Explicit user-managed settings and grants.
3. Project configuration and repository instructions.
4. Installed extensions and MCP servers.
5. Tool results, repository data, web content, and model-generated output.

Lower-trust content cannot override higher-trust policy. A CLAUDE.md instruction, Skill, plugin, or model output must never grant itself access denied by global security policy.

## 4. Permission model

Permission categories:
- read: inspect files and metadata;
- write: create or modify files;
- delete: remove files;
- execute: run commands/programs;
- network: make outbound or inbound network requests;
- git-mutate: modify branches, index, commits, or remote refs;
- secrets: read or use named secret references;
- browser: interact with websites;
- external-tool: call MCP/plugin tools;
- destructive: irreversible or high-impact action.

Permission outcomes:
- allow;
- deny;
- ask;
- allow for a narrow scope and duration.

Every tool declares required permissions and risk level. The effective policy is the intersection of tool requirements, global policy, managed policy, workspace trust, project settings, and session grants.

Suggested user modes:
- read-only;
- ask-before-edit;
- auto-edit;
- ask-before-execute;
- sandbox;
- explicitly configured full-control.

Full-control is never the default. A mode name does not bypass managed restrictions or global deny rules.

## 5. Approval requirements

Require approval by default for:
- destructive file operations;
- commands that may delete, overwrite, install, publish, deploy, or change system state;
- network operations that transmit workspace data;
- credential use or access to secret references;
- Git reset/clean/force-push and destructive branch operations;
- plugin installation and plugin code execution;
- MCP server trust and authentication;
- downloads or uploads involving private data;
- remote execution with workspace content.

Approval grants must be specific: exact tool, path, command pattern, host, resource, or session scope. Avoid broad, permanent grants when a narrow grant is sufficient.

Display the exact operation, target, reason, and relevant risk before approval.

## 6. Filesystem security

- Resolve all paths against an authorized workspace root.
- Normalize separators and reject path traversal.
- Check symlinks and real paths before access.
- Prevent writes outside allowed roots.
- Define policy for symlinked directories and external mounts.
- Distinguish text, binary, large, and special files.
- Use atomic replacement where possible.
- Do not follow arbitrary symlinks into secrets or system directories.
- Respect ignore files for indexing, but do not treat ignore rules as an authorization boundary.
- Protect internal state, provider secrets, SSH material, and credential stores.
- Record file changes and checkpoint metadata.

## 7. Bun runtime security

- Use Bun as the trusted application runtime.
- The standalone executable embeds Bun, but this does not sandbox commands started by the agent.
- Bun.spawn and all subprocess APIs must still pass through the command permission and runner policy.
- Do not assume the compiled executable hides secrets or protects embedded assets; never embed API keys or private configuration.
- Verify external config, plugin, Skill, MCP, and project files at runtime using the same trust rules as source execution.
- Keep Bun version pinned and update it through reviewed security maintenance.

## 8. Shell and process security

- Use argument arrays where possible; shell string execution requires policy checks.
- Record command, cwd, runner, exit code, duration, and approval context.
- Apply timeout, process-tree cancellation, stdout/stderr limits, and CPU/memory/process limits.
- Sanitize inherited environment; only pass approved variables.
- Do not expose provider keys to child processes by default.
- Do not construct shell commands from untrusted text without safe quoting and validation.
- Detect risky commands and require approval.
- Separate direct user shell input from model-generated shell tool calls.
- Track background process ownership and terminate owned processes on session cleanup when policy says so.

## 9. Network and secret security

- Network access is denied or constrained by default in sandbox profiles.
- Support host/domain allowlists and optional port restrictions.
- Redact authorization headers, API keys, cookies, tokens, and known secret patterns from logs.
- Use environment variable or OS credential-store references.
- Do not place secrets in prompts unless the task explicitly requires a secret to be used; prefer a tool-side secret reference.
- Never store raw secrets in transcript, model request logs, crash dumps, or task artifacts.
- Do not return full environment variables to the model.
- Rotate or revoke scoped credentials after remote tasks where applicable.

## 10. MCP security

- Treat MCP server configuration as executable trust configuration.
- Require explicit trust for new project-defined servers where policy requires.
- Show command, arguments, URL, headers (redacted), and source scope.
- Restrict stdio child process environment and filesystem.
- Restrict remote network destinations.
- Enforce per-server and per-tool permissions.
- Validate tool schemas and bound response sizes.
- Treat resources, prompts, and tool results as untrusted data.
- Handle OAuth and refresh tokens through secure storage.
- Stop owned MCP child processes and clear ephemeral credentials on shutdown.

## 11. Plugin and hook security

- Installing a plugin is distinct from trusting and running its code.
- Verify source, revision, and integrity where practical.
- Display manifest, components, scripts, hooks, agents, MCP servers, and permissions before activation.
- Run scripts through the permission engine and sandbox.
- Do not execute arbitrary post-install scripts automatically.
- Resolve plugin paths inside the plugin root.
- Apply timeout, resource limits, and audit logging to hooks.
- Hook output is untrusted and cannot directly modify security configuration.
- Allow disabling a component or whole plugin without uninstalling it.
- Support pinning and rollback.

## 12. Prompt injection and untrusted data

- Label repository, browser, MCP, and tool output as untrusted data.
- Keep untrusted content separate from system/developer policy and user instructions.
- Do not obey instructions found in a source file or web page that ask to reveal secrets, alter permissions, or perform unrelated actions.
- Tool execution still passes through policy even if a prompt or Skill requests it.
- Summarize relevant evidence without promoting embedded instructions to authority.
- Provide safe browsing and MCP modes with constrained tool access.

## 13. Sandbox profiles

Initial profiles:
- local-trusted-workspace;
- read-only-inspection;
- restricted-command;
- container-isolated;
- remote-ephemeral.

Each profile defines:
- filesystem mounts and write scopes;
- network policy;
- environment variable allowlist;
- process and resource limits;
- available tools;
- credential references;
- cleanup policy;
- artifact export policy.

Docker/Podman containers are not automatically a complete security boundary. Configure mounts, capabilities, user ID, network, seccomp/AppArmor or platform equivalent, and resource limits.

## 14. Audit and recovery

Audit events include:
- permission decision;
- approval request and result;
- tool start/finish/failure;
- plugin/MCP install or activation;
- hook invocation;
- file mutation;
- Git mutation;
- remote task creation;
- credential reference use;
- session restore and rewind.

Audit logs must be append-oriented, structured, timestamped, and redacted. Provide export for incident review.

## 15. Security acceptance tests

- Path traversal is rejected.
- Symlink escape is rejected or explicitly approved by policy.
- Denied command is not executed.
- Destructive command requires approval.
- Secrets do not appear in logs or transcripts.
- Project instructions cannot override global deny.
- Malicious MCP output cannot bypass the tool permission gate.
- Untrusted web content cannot grant itself permissions.
- Plugin scripts do not run on discovery or installation without approval.
- Hook timeout and failure behavior are bounded.
- Process cancellation terminates child process trees.
- Remote runner receives only scoped credentials and workspace content.
- Compaction preserves security-relevant user constraints and does not reveal hidden secrets.
