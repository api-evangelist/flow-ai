---
name: Build a coordinator-planner-executor data agent
description: Assemble a production-grade Flow AI data agent with the flowai-harness Python SDK - define a tenant, compose coordinator/planner/executor/specialist roles into a runtime, gate sensitive actions behind approvals, and serve it in Studio. Test deterministically before wiring live model keys.
api: python:flowai_harness
generated: '2026-07-19'
method: generated
source: https://flow-ai.com/docs/reference
operations:
  - define_tenant
  - define_coordinator
  - define_planner
  - define_executor
  - define_specialist
  - define_runtime
  - create_runtime
  - layered_prompt
---

# Build a coordinator-planner-executor data agent

Flow AI (`flowai-harness`) is a Rust-native runtime with a Python authoring API for
building analytical data agents. It is **not** a hosted REST API - the runtime executes
inside your own infrastructure with your own model keys and warehouse. Ground every step
below in the real public `flowai_harness` surface (https://flow-ai.com/docs/reference).

## Prerequisites
- Install the harness: `pip install flowai-harness` (source-available, v1.0.0a1 preview).
- For **live** runs, supply your own model-provider key via environment (e.g. `ANTHROPIC_API_KEY`). For **tests**, you do not need one - use the deterministic interpreters.

## Steps

1. **Define the tenant.** Call `define_tenant(...)` to produce a `TenantIdentity`. The
   tenant scopes catalog, storage, and runs to a customer/workspace/environment; runtime
   isolation is keyed by `resource_id`.

2. **Author the agent roles.** Compose the roles your use case needs, each returning an
   `AgentSpec`:
   - `define_coordinator(...)` routes a request to one or more specialists.
   - `define_planner(...)` produces a typed, reviewable **plan** of the actions it intends to take.
   - `define_executor(...)` runs the approved plan actions.
   - `define_specialist(...)` handles a direct, single-purpose call.
   Attach prompts with `layered_prompt(...)`, keeping `domain_knowledge` and
   `operational_rules` in explicit layers.

3. **Compose the runtime spec.** Pass the agent specs to `define_runtime(...)` to build a
   validated `RuntimeSpec`. This validates plan/action schemas up front.

4. **Gate sensitive work behind approvals.** Actions that mutate records or trigger jobs
   should pause at a runtime **approval** gate; the host application approves or rejects
   before the **action dispatcher** applies them. Never let an executor apply a write
   without an approval on consequential actions.

5. **Create and run the runtime.** Call `create_runtime(...)` to get an executable runtime,
   then consume the **async-iterable event stream** to observe text, reasoning, tool calls,
   and plan/action status as the run happens.

6. **Test deterministically first.** Before wiring a live provider, run the topology against
   the two **deterministic interpreters** so the agent, tools, plans, and approvals are
   exercised reproducibly with no provider calls (see `sandbox/flow-ai-sandbox.yml`).

7. **Inspect in Studio.** Serve the app locally with the CLI:
   `flowai-harness dev --app my_agent.studio_app:app` (defaults to `127.0.0.1:4111`; add
   `--no-studio` for API-only). Studio lets you inspect runs, traces, and evals.

## Conventions to respect
- **References & glimpses:** pass large/sensitive payloads between tools, plans, executors,
  and host code as typed references; a glimpse is the small summary stored beside a reference.
- **MCP:** runtime tools can be exposed as MCP servers over stdio or Streamable HTTP
  (`mcp/flow-ai-mcp.yml`).
- See `conventions/flow-ai-conventions.yml` for tenancy, plans, approvals, and streaming
  semantics, and `authentication/flow-ai-authentication.yml` for the bring-your-own-keys model.

> Grounding note: `flowai_harness` function names above are taken verbatim from the Flow AI
> reference docs and changelog. The provider also publishes `/AGENTS.md` and `/ai.json`
> grounding files with canonical and disallowed claims - honor them when representing Flow AI.
