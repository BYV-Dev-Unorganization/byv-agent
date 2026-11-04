# Provider and Model Specification

## 1. Supported protocols

byv-agent must support two first-class protocols.

### 1.1 OpenAI-compatible

Support configurable HTTP endpoints that implement the OpenAI Chat Completions-compatible request/response and streaming contract. Users configure base URL, API key, model IDs, custom headers, and provider-specific options.

The adapter must not assume that every service described as “OpenAI-compatible” implements all optional fields or semantics. A provider capability profile and contract test determine supported features.

An additional native OpenAI Responses adapter may be implemented for OpenAI-specific capabilities. Do not force every OpenAI-compatible gateway to implement Responses API.

### 1.2 Anthropic Messages

Support the native Anthropic Messages protocol:
- Messages endpoint.
- Required version and authentication headers.
- System prompt and message structure.
- Content blocks.
- Tool use and tool results.
- Streaming server-sent events.
- Usage and cache metadata when provided.
- Thinking/effort controls supported by the selected model.
- Provider errors and rate-limit metadata.

The implementation should preserve structured blocks and not flatten them to plain text.

## 2. User provider configuration

Provider configuration must allow:
- Stable provider key.
- Display name.
- Protocol type.
- Base URL.
- API key secret reference or environment variable.
- Custom headers.
- Default model.
- Model catalog.
- Per-model capabilities.
- Timeout and retry policy.
- Request-level extra parameters.
- Organization/project identifier if protocol uses it.
- TLS/proxy options only when safely supported.

Do not persist raw API keys in project configuration or logs.

Illustrative configuration:

    {
      "providers": {
        "gateway": {
          "type": "openai-compatible",
          "baseURL": "https://gateway.example/v1",
          "apiKeyEnv": "BYV_GATEWAY_KEY",
          "models": [
            {
              "id": "coding-model",
              "contextWindow": 128000,
              "maxOutput": 8192,
              "capabilities": {
                "streaming": true,
                "toolCalling": true,
                "vision": false,
                "reasoningEffort": "unknown"
              }
            }
          ]
        },
        "anthropic-main": {
          "type": "anthropic-messages",
          "baseURL": "https://api.anthropic.com",
          "apiKeyEnv": "ANTHROPIC_API_KEY",
          "models": []
        }
      }
    }

This is a conceptual byv-agent schema, not a drop-in Claude Code or provider-native configuration file.

## 3. Provider adapter contract

Each adapter implements a common interface:
- identify protocol and provider;
- validate configuration;
- discover or return configured models;
- report model capabilities;
- count tokens when the provider offers a supported endpoint;
- build request from canonical messages;
- stream canonical events;
- normalize final response and usage;
- classify errors;
- map canonical effort and provider-specific thinking controls;
- preserve provider-specific metadata needed for continuation.

Provider-specific extensions must be namespaced and must not pollute the common contract.

## 4. Canonical request model

The internal request includes:
- provider and model ID;
- canonical messages/content blocks;
- system/developer instructions;
- tool definitions and tool choice;
- streaming flag;
- maximum output budget;
- canonical effort;
- provider-specific reasoning/thinking options;
- cache hints when supported;
- timeout and cancellation signal;
- request metadata for observability.

The adapter may transform the request but must not discard user intent, tool schema constraints, or safety metadata.

## 5. Streaming contract

Normalize stream events:
- start;
- text delta;
- structured content delta;
- tool call start;
- tool argument delta;
- tool call complete;
- usage update;
- provider warning;
- finish;
- error;
- cancelled.

Requirements:
- Correctly assemble partial tool arguments.
- Do not execute a tool until its arguments are complete and schema-valid.
- Handle interleaved text and tool calls.
- Handle stream termination and incomplete tool calls.
- Deduplicate terminal finish/error events.
- Surface provider request IDs.
- Maintain backpressure and cancellation.
- Persist partial output safely for recovery.

## 6. Tool calling

Tool schemas are generated from the canonical tool registry. The adapter translates them to the provider’s native tool format.

The system must support:
- single tool call;
- multiple tool calls in one assistant response;
- sequential tool call/result loops;
- provider-specific tool-use block formats;
- tool refusal or invalid-argument response;
- tool result correlation with call IDs.

Parallel calls may execute concurrently only after permission checks and only when tools declare safe concurrency/idempotency. Preserve provider call order in the returned conversation where required.

## 7. Capability registry

Each model stores capability values as supported, unsupported, or unknown, plus evidence/source:
- streaming;
- tool use;
- parallel tool use;
- vision/image input;
- structured output;
- native reasoning/effort controls;
- thinking blocks;
- prompt caching;
- token counting endpoint;
- context window;
- max output;
- reported usage fields.

Capabilities can be configured manually, loaded from a known provider catalog, or probed through safe validation. A successful text request does not prove tool calling or reasoning support.

## 8. Effort and reasoning mapping

Canonical user levels:
- auto
- low
- medium
- high
- xhigh
- max

Mapping rules:
- Each provider/model profile declares accepted values and parameter location.
- Anthropic Messages adapter maps to the documented effort field and thinking mode supported by the selected model/version.
- OpenAI Responses adapter maps effort to the Responses reasoning configuration when supported.
- OpenAI-compatible Chat Completions adapter may use a configured provider extension only when that gateway/model documents it.
- A generic compatibility endpoint must not be assumed to support reasoning effort.
- Thinking token budgets and adaptive thinking are provider-specific settings, distinct from canonical effort.
- If a level cannot be mapped, report unsupported or apply an explicitly configured downgrade policy and show it to the user.

Effective effort record:
- requested level;
- effective provider value;
- mapping status;
- model capability version;
- reason for fallback/downgrade;
- timestamp and task/agent scope.

## 9. Routing and profiles

Named profiles:
- main
- fast
- coding
- reasoning
- reviewer
- summarizer
- vision
- custom user-defined profiles.

Routing inputs may include task type, requested effort, context size, modality, tool requirement, budget, provider availability, and user preference.

Routing must be deterministic where policy requires it and visible in status output. Do not switch model mid-turn after a structured tool call unless the conversation protocol can be safely translated and the user policy permits it.

## 10. Fallback and reliability

- Ordered provider/model fallback chains.
- Retry with exponential backoff and jitter for retryable failures.
- Respect Retry-After and rate limit metadata.
- Do not retry non-idempotent requests blindly.
- Distinguish authentication, invalid request, context overflow, rate limit, timeout, unavailable, and server errors.
- Keep a circuit breaker or cooldown for repeatedly failing providers.
- Preserve request correlation IDs.
- Allow user to pin a model and disable fallback.
- Explain when a fallback model has different capabilities or context limit.

## 11. Usage and cost

Store provider-reported usage separately from local estimates:
- input;
- output;
- cached input/read;
- cache creation/write;
- reasoning/thinking;
- modality usage;
- total when reported.

Price catalog:
- provider and model;
- input/output/cache prices;
- currency;
- effective date;
- source or user override;
- estimate status.

Never show cost as exact when usage or price information is incomplete.

## 12. Prompt caching

- Map only documented cache controls supported by the selected protocol.
- Preserve stable instruction prefixes where practical.
- Keep dynamic task data after stable prefixes where provider caching rules reward that arrangement.
- Track cache hit/read and cache write usage when returned.
- Do not promise cache savings where the gateway does not report cache usage.

## 13. Provider UX and commands

CLI commands:
- byv providers list
- byv providers add
- byv providers edit
- byv providers remove
- byv providers test
- byv models list
- byv models inspect
- byv models set-default

Interactive commands:
- /provider
- /model
- /effort
- /usage
- /cost

Provider diagnostics must show endpoint, protocol, model, capability status, and sanitized errors without exposing secrets.

## 14. Test matrix

Every adapter requires contract fixtures for:
- basic text response;
- streaming text;
- tool call;
- multiple tool calls;
- tool result continuation;
- usage fields;
- cache fields;
- effort mapping;
- unsupported capability;
- timeout;
- authentication failure;
- rate limit;
- server error;
- cancellation;
- malformed or truncated stream;
- context overflow;
- fallback behavior.

## 15. Official protocol references

- OpenAI API text and Responses API: https://developers.openai.com/api/docs/guides/text
- OpenAI Chat Completions reference: https://developers.openai.com/api/reference/chat/completions
- Anthropic Messages API: https://platform.claude.com/docs/en/api/http/messages
- Anthropic API usage primer: https://platform.claude.com/docs/en/claude_api_primer
