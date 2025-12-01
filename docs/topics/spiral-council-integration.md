# Integrating Spiral Council with A2A

Spiral Council can reuse the Agent2Agent (A2A) protocol to interoperate with external agents while keeping its own orchestration patterns. The following steps outline how to move from alignment to implementation.

## 1) Establish the baseline
- Inventory current Spiral Council agents, their roles, and the message formats they exchange today.
- Decide which agents should expose A2A **servers** (to be invoked) versus **clients** (invoking external agents) or both.
- Map existing security posture (identity, authN/Z, network controls) to the A2A transport so you know which authentication schemes to expose.

## 2) Model Agent Cards first
- Draft Agent Cards for each exposed agent using the fields in [specification §5](../specification.md#5-agent-discovery-the-agent-card) (identity, `url`, `capabilities`, `skills`, `authentication`).
- For Council-specific capabilities, capture them as `AgentSkill` entries with concrete `inputModes`/`outputModes` and examples.
- Decide where Agent Cards live: well-known path for public agents, curated registry for internal agents, or static configuration for tightly coupled pairs.

## 3) Choose interaction patterns
- Decide which skills require **synchronous** requests versus **streaming** (`resultStream`) or **async push** (`resultPush`).
- If Spiral Council already emits webhooks/events, align them with `pushUrl` semantics so external agents can receive results without polling.
- Standardize error handling and retries around JSON-RPC 2.0 so Spiral Council and partners share failure semantics.

## 4) Authentication and trust
- Pick default auth schemes (e.g., OAuth2 bearer, mTLS) and note them in the Agent Card’s `authentication.schemes`.
- For sensitive agents, serve authenticated extended cards ([spec §7.10](../specification.md#710-agentgetauthenticatedextendedcard)) so capabilities are only revealed to trusted peers.
- Set a policy for key/certificate rotation and limit the lifetime of any static secrets.

## 5) Build and test
- Use an official SDK (Python/JS/Java/.NET/Go) to scaffold a proof-of-concept A2A server + client that exercises one Spiral Council skill end-to-end.
- Validate streaming/push flows with a sample task to ensure tokenization and back-pressure behave as expected in Council’s runtime.
- Log JSON-RPC envelopes and correlate them with internal spans to keep observability consistent.

## 6) Hardening and rollout
- Add conformance checks to CI (linting Agent Cards, contract tests for required skills, authentication enforcement).
- Create a minimal registry or config service so Council agents can discover each other and external partners without manual URLs.
- Publish a runbook that documents expected behaviors, timeouts, and recovery steps for each supported interaction mode.

## Suggested deliverables
- Approved Agent Card templates for each Spiral Council agent category.
- A working POC showing one Council agent invoking and being invoked over A2A.
- CI job that validates Agent Cards and exercises at least one synchronous, streaming, and push flow.
