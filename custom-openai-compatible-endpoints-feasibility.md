# Custom OpenAI-Compatible Endpoint Support in Warp (Feasibility Notes)

## What exists today

Warp already supports Bring-Your-Own-Key (BYOK) for multiple providers through the API key payload that is attached to AI requests.

- Client-side key storage and request plumbing currently includes:
  - `openai`
  - `anthropic`
  - `google`
  - `open_router`
  - optional AWS Bedrock credentials
- These are sent to `warp_multi_agent_api::request::settings::ApiKeys` when BYOK is enabled.

## Key limitation relevant to "custom OpenAI-compatible endpoints"

There is currently no client-side setting or request field for a custom provider base URL (or equivalent endpoint override) in the AI request path.

In other words:

- keys are configurable,
- provider identity is represented,
- but endpoint host/URL is not represented in `RequestParams`/request settings from this client.

So custom OpenAI-compatible endpoints are **not currently supported in this client codepath**.

## Why this appears tractable (not inherently huge)

The current architecture already has:

- provider-specific BYOK fields,
- provider-aware error handling (including OpenRouter in invalid-key handling),
- centralized request construction for multi-agent generation.

That suggests a relatively direct extension:

1. Add endpoint override fields to client settings + secure storage model (likely alongside BYOK keys).
2. Extend the API contract (`warp_multi_agent_api::request::settings`) to include provider endpoint overrides.
3. Add UI/settings surface to edit endpoint(s), with validation.
4. Update server routing logic to honor endpoint override when provider is OpenAI-compatible and BYOK is enabled.
5. Add guardrails (TLS, allowed schemes, optional workspace/admin policy).

## Minimal implementation shape (proposal)

A minimal first cut could target only OpenAI-compatible routing and avoid broad provider generalization:

- Add optional fields:
  - `openai_base_url: Option<String>`
  - (optional later) `openai_organization`, extra headers, etc.
- Keep model selection unchanged.
- When `openai_base_url` is present, server uses same OpenAI protocol payloads against that base URL.
- If unsupported model/provider pairing occurs, return an explicit validation error.

## Risks / details to resolve

- API compatibility drift across OpenAI-compatible vendors (tool calling, streaming/event shape, reasoning fields).
- Auth style differences (Bearer key vs custom header).
- Model catalog mismatch (names may differ from OpenAI defaults).
- Enterprise policy requirements (disallow unknown hosts).
- Telemetry and redact-safe logging for custom hostnames.

## Suggested rollout

1. Feature-flagged internal/dev rollout.
2. BYOK + custom endpoint for OpenAI provider only.
3. Add compatibility matrix + error UX.
4. Expand to custom Anthropic-compatible / Gemini-compatible only if demanded.

