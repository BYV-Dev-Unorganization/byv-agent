# byv-agent Configuration Specification

## 1. Configuration goals

- Keep byv-agent-owned settings separate from Claude Code compatibility files.
- Support user, project, project-local, managed, environment, and CLI/session scopes.
- Validate all settings with a versioned schema.
- Show effective settings and source provenance.
- Never reveal secret values in diagnostics.
- Keep secrets out of committed project files and transcripts.

## 2. Directory layout

Global state:

    ~/.byv-agent/
      config/
        settings.json
        providers.json
        models.json
        pricing.json
      sessions/
      memory/
      logs/
      cache/
      plugins/
      marketplaces/
      checkpoints/
      artifacts/
      state/

Project state and compatibility:

    project/
      CLAUDE.md
      AGENTS.md
      .mcp.json
      .claude/
        settings.json
        settings.local.json
        rules/
        skills/
        agents/
        commands/
      .byv-agent/
        config.json
        tasks/
        local/

Do not commit credentials, session transcripts, machine-local state, task artifacts, or project-local secrets. The project-local byv-agent directory may contain only explicitly shareable configuration and task metadata.

## 3. Configuration scopes and precedence

Recommended effective precedence, highest to lowest:
1. Managed policy that is explicitly marked non-overridable.
2. Explicit CLI flag or current-turn/session override.
3. Project-local configuration.
4. Shared project configuration.
5. User configuration.
6. Built-in defaults.

Security is not a simple last-write-wins merge. A lower-trust scope cannot weaken a higher-trust deny, sandbox, secret, or network restriction.

For each setting define one merge strategy:
- scalar: highest-precedence value wins;
- ordered fallback chain: entire list comes from the highest-precedence source defining it;
- allow/deny lists: apply documented union/intersection semantics;
- hook lists: merge only according to the compatibility contract;
- provider/model profiles: merge by stable identifier with validated field overrides.

The exact Claude Code settings behavior is documented in the compatibility specification. byv-agent-native keys should use a distinct namespace or its own configuration file.

## 4. settings.json schema areas

Suggested areas:
- version: config schema version;
- defaultProvider and modelProfiles;
- effortLevel and routing;
- permissionMode and permissionRules;
- context and compaction;
- session and memory;
- token and cost budgets;
- terminal and process execution;
- workspace and ignore rules;
- tools and MCP policy;
- plugin/skill/agent loading;
- hooks;
- background jobs and scheduler;
- logging and telemetry;
- UI and status line.

Example:

    {
      "version": 1,
      "defaultModelProfile": "main",
      "effort": {
        "default": "medium"
      },
      "context": {
        "autoCompact": {
          "enabled": true,
          "trigger": {
            "type": "percent",
            "value": 80
          },
          "target": {
            "type": "percent",
            "value": 60
          },
          "preserveRecentTurns": 8
        }
      },
      "budgets": {
        "maxTurns": 80,
        "maxToolCalls": 300,
        "maxSessionCost": null
      },
      "telemetry": {
        "enabled": false
      }
    }

The values are examples, not universal defaults. Validate ranges, reject unknown critical values, and migrate schema versions explicitly.

## 5. Provider and model configuration

Provider details belong in providers.json or a secure provider store. Model profiles belong in models.json or the provider catalog.

Allow:
- protocol;
- endpoint;
- secret reference;
- custom headers;
- model identifier;
- capability metadata;
- timeout;
- retry policy;
- default effort mapping;
- price metadata reference.

Environment variables may use a documented reference syntax. Resolve values at runtime, redact them in logs and never interpolate them into project prompt content.

## 6. Secrets

Preferred options:
1. OS credential manager/keychain.
2. Encrypted local secret store.
3. Environment variables for simple deployments.

Never:
- commit API keys;
- store plaintext keys in repository settings;
- print raw Authorization or API-key headers;
- include secrets in diagnostic bundles;
- return all environment variables to the model.

## 7. Configuration commands

- byv config list
- byv config get <key>
- byv config set <key> <value>
- byv config unset <key>
- byv config validate
- byv config sources
- byv config effective

The effective view must redact secret values and identify the source scope for every setting.

## 8. Validation and migration

- Validate JSON and YAML.
- Include JSON path and source file in errors.
- Preserve unknown compatibility metadata where required.
- Reject invalid security policies.
- Provide explicit migration steps.
- Back up configuration before automatic migration.
- Do not overwrite hand-edited settings without user consent.
- Include schema version in generated files.
