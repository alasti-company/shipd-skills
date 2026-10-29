---
name: shipd-rollout
description: Inspect, target, publish, or disable shipd flags through MCP for an explicitly scoped rollout. Use when enabling a feature for a cohort, publishing flag drafts, checking approvals, or reverting flag exposure.
---

# Operate a shipd flag rollout

Use the connected shipd MCP tools when available. Discover their actual schemas; tool names can have a client-specific prefix. The reference is https://docs.useshipd.com/reference/mcp and connection setup is https://docs.useshipd.com/operations/connect-an-agent. If the connection is missing, give setup instructions and continue local preparation; do not pretend a remote change occurred.

## Establish the target and change

Resolve organization, project, flag, and environment from the user's request and resource reads (`project_list`, `environment_list`, `flag_list`, `flag_get`, `flag_listEnvironmentStates`). Environment production status matters, not just its name. Reuse an existing flag when appropriate. Resolve ambiguous targets before writes while continuing read-only investigation.

Read the current definition and environment state. Identify the intended audience, fallthrough, enabled state, rule ordering, and any existing draft. Preserve unrelated rules and do not publish someone else's draft as a side effect. Use stable evaluation identities for sticky percentage targeting; consult https://docs.useshipd.com/flags/percentage-rollouts before constructing a rollout rule. Do not assume an enabled flag serves true: targeting and fallthrough determine the result.

Use the user's existing authorization for the requested environment and exposure. A request to implement a feature alone does not authorize enabling it in production. Honor shipd's permissions and approval policies; do not change production status or policies to get around a rejected operation.

## Save, publish, verify

Use the appropriate discovered tools, such as `flag_updateEnvironment` or `flag_toggleEnvironment`. Read before updating and supply the environment-state `expectedVersion`, not the flag definition version. Definition updates through `flag_update` use the definition version and affect every environment.

Every logical mutation needs a unique printable, space-free `idempotencyKey`. Retry uncertain requests with the same key and identical input. A stale version requires a fresh read, a revised proposal, and a new key; do not blindly replay an old patch over new state. If an outcome remains in progress, reconcile state and `mcp_history` before another mutation. Replay records last seven days; reconcile before retrying beyond that window.

For saved drafts, reread `flag_listEnvironmentStates` and call `flag_publishEnvironments` for only the intended environments, supplying each current `expectedVersion` and an idempotency key. Saving a draft is not publication. A `pending_approval` outcome has not applied that protected change: return the dashboard review link and use `mcp_approvalStatus` when checking its status. MCP cannot approve the request.

Read back the environment state after an applied change and distinguish saved, pending, and published state. SDK propagation is asynchronous through snapshots, streaming, and polling; a successful control-plane write alone does not prove application exposure.

## Observe and revert within scope

Use available `evaluationFact_list`, `evaluationFact_byRule`, and `evaluationFact_projectSummary` for evaluation diagnostics. Evaluation counts do not establish application health or product success. If Sentry, Axiom, PostHog, or another source is connected and relevant, inspect signals for the correct environment, deployed version, time window, and exposed cohort.

Advance only within the user's authorized stages and health criteria. Without sufficient telemetry, report the gap and hold further progression. This skill does not supply a durable background controller; do not promise monitoring after the agent session ends.

For an authorized rollback, restore the known control behavior for the intended environment using current versions and the same publication/approval process. Confirm what disabling the flag actually serves before choosing it. Flag rollback changes exposure; it does not undo migrations, writes, or external side effects.

Report the exact target, before/after exposure, publication or approval state, verification evidence, and remaining rollout or rollback steps. Never report a rollout healthy based only on a successful publish.
