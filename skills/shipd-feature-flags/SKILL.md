---
name: shipd-feature-flags
description: Implement features behind shipd flags, including definitions, typed SDK evaluation, and fallback behavior. Use when adding a feature to an app that uses shipd or when asked to introduce shipd feature flags.
---

# Implement a feature with shipd

Deliver the requested feature behind a flag with a working existing-behavior path. A coding request authorizes local implementation; use the user's release instructions to determine whether remote changes are also in scope.

## Discover the integration

Inspect the app's flag definitions, generated types, SDK initialization, and nearby flag checks. Reuse its existing client and conventions. Locate `shipd.config.ts` or `shipd.config.go` and `.shipd/config.json`; in a monorepo, identify the owning application before choosing a config. Do not replace an existing flag provider without being asked.

For a new integration, use the SDK appropriate to the runtime. TypeScript uses `shipdit`, definitions import `defineFlags` from `shipdit/config`, and the CLI package is `@shipdit/cli`. Go uses `github.com/alasti-company/useshipd.com/sdks/go` and its `config` package. Read the relevant SDK documentation before choosing framework-specific APIs: https://docs.useshipd.com/sdks/node, /sdks/react, /sdks/nextjs, /sdks/browser, or /sdks/go.

## Define and gate

Add a descriptive flag key to the existing definitions. For a boolean feature release, start with `default: false` and preserve the old behavior when false. For variants, use an explicit control value. A TypeScript definition looks like:

```ts
import { defineFlags } from "shipdit/config";

export default defineFlags({
  "new-checkout": {
    type: "boolean",
    default: false,
    description: "Enable the redesigned checkout flow",
  },
});
```

Merge into the existing config; do not replace other definitions. Generate types with the project's CLI runner (`shipd types gen`); do not hand-edit generated files. Respect configured output paths.

Gate the behavior at the appropriate boundary, including backend work when needed. Hiding a button alone does not gate a backend change. Flags do not replace authorization. Reuse a long-lived SDK client; avoid constructing clients per render or request. Use the application's existing identity context so targeting is consistent. Keep management credentials and server SDK keys out of browser bundles; browser SDKs require client keys.

Keep runtime fallback values consistent with the safe control behavior. SDK evaluation is local against a snapshot; a network failure can retain the last snapshot, so a configured fallback is not a remote kill switch. Follow the SDK's readiness and shutdown patterns for the runtime.

## Verify and hand off

Verify both feature branches and fallback behavior with tests appropriate to the change. Run type checks and review generated output. If remote read access is available, inspect drift for the explicit project and environment before proposing synchronization. Do not invent project IDs, credentials, or successful remote operations.

Report the flag key, config and evaluation locations, fallback behavior, validation, and whether the flag exists only locally, is saved remotely, is a draft, is pending approval, or is published. Include the remaining activation step. Do not imply that merging or deploying code activates a flag.

For synchronization semantics: https://docs.useshipd.com/config-as-code/sync. Definition changes affect all environments; environment fallthrough and targeting require separate attention.
