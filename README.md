# shipd agent skills

Agent workflows for implementing features behind shipd flags, syncing flag
configuration through CI, and managing rollout through shipd MCP.

## Install

Run from your application repository and select the skills and agent you use:

```bash
npx skills add alasti-company/shipd-skills
```

## Skills

| Skill | Purpose | Example prompt |
| --- | --- | --- |
| [shipd-feature-flags](skills/shipd-feature-flags/SKILL.md) | Define flags, generate types, and gate application behavior with safe fallbacks. | Implement the new checkout behind a shipd flag. |
| [shipd-ci](skills/shipd-ci/SKILL.md) | Validate flag definitions and synchronize configuration through CI. | Set up CI to sync our shipd flags on merge. |
| [shipd-rollout](skills/shipd-rollout/SKILL.md) | Target cohorts, publish drafts, check approvals, and revert exposure. | Enable new-checkout for our internal users in production. |

## Connect shipd

Skills supply instructions. Remote flag operations also require an authenticated
shipd connection with access to the intended organization and project.

Connect your agent to `https://mcp.useshipd.com/mcp` using the
[agent setup guide](https://docs.useshipd.com/operations/connect-an-agent).
For CLI-based synchronization, follow the
[CI setup guide](https://docs.useshipd.com/config-as-code/ci).
Never commit credentials to this repository or to skill files.

Local implementation, saving drafts, publishing configuration, and enabling a
feature are separate operations. These skills respect existing production
permissions and approval policies. Rollout guidance runs within the agent session;
it does not provide a background monitoring service.

## Contributing

Edit the relevant `skills/<name>/SKILL.md`. Keep instructions grounded in the
[shipd documentation](https://docs.useshipd.com), preserve YAML frontmatter with
`name` and `description`, and verify command and MCP contracts before changing
operational guidance.

## Using this repository as a submodule

```bash
git submodule add https://github.com/alasti-company/shipd-skills.git skills
```

After cloning a repository that includes the submodule:

```bash
git submodule update --init --recursive
```

To update its pinned revision, fetch the desired commit inside the submodule,
then commit the updated submodule pointer in the parent repository.
