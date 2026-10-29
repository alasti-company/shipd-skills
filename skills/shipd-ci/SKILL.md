---
name: shipd-ci
description: Set up or repair CI for shipd flag validation and config synchronization using the shipd CLI. Use when publishing flags on merge or checking flag drift in pull requests.
---

# Sync shipd flags through CI

Inspect the repository's CI provider, package manager, flag config, generated output, and existing deployment ordering. Extend its workflow rather than adding a parallel release system. Use the installed or pinned `@shipdit/cli`; verify command support with its help. Do not invent a `shipd ci` command: the supported workflow uses `types gen`, `diff`, and `push`.

Read https://docs.useshipd.com/config-as-code/ci and https://docs.useshipd.com/reference/cli for provider examples and current contracts. Follow these invariants:

- Select the intended project and environment explicitly. Set `SHIPD_API_URL` for hosted CI; an unconfigured runner otherwise defaults to localhost. Authenticate using service account tokens in CI secrets through `SHIPD_API_KEY`.
- Use separate read-only PR and write-on-main credentials. Tokens for push must be project-bound, not environment-bound, because definitions change project-wide. Use the documented scopes; production changes require production permission even for default-only changes.
- Flag configs execute code. Do not expose secrets to untrusted PR code, dependencies, or config imports. Use the provider's trusted review/environment gate before a credentialed job; do not use privileged fork workflows to execute PR code.
- Generate types and check that the configured generated output is tracked and unchanged. Run relevant application checks. Watch config dependencies as well as the entrypoint in path filters.
- Show `shipd diff --exit-code --project <project> --env <environment>` on PRs. Exit 0 means clean, 2 means expected drift that needs review, 4 means unlinked flags, and 1 means failure. Choose deliberately whether planned drift blocks merging; do not silently treat every nonzero result as success.
- On the authorized protected branch, serialize pushes per project/environment and use `shipd push --ci --json --project <project> --env <environment>`. Do not add `--force` or `--prune` as routine automation.
- Exit 3 is a conflict: fail and reconcile remote changes, preserving local edits before any pull. Exit 5 means some changes await approval and other changes may already have applied. Report that partial state explicitly. Inspect JSON `drafts`, `approvalRequests`, and `outcome` even on exit 0: saved targeting drafts are not published.

Publishing configuration and deploying application code are separate events. Keep new release flags on their safe control value until the required code is deployed. CLI push can leave targeting drafts requiring publication through the dashboard or MCP; snapshot publishing is not a substitute for publishing those drafts. Never label a release live solely because the CI job passed.

Validate workflow syntax, shell exit handling, trigger paths, credential boundaries, and generated output checks without writing to a live environment just to test the workflow. Report files changed, required secret names and scopes, publication behavior, and any pending setup. Do not create credentials or change production configuration unless the task authorizes it.
