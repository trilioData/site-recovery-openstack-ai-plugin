# site-recovery-openstack-ai-plugin

The `site-recovery-openstack` agent plugin for
[Trilio Site Recovery for OpenStack](https://docs-tsr.trilio.io/trilio-site-recovery-for-openstack),
owned by the TSR-O team. It is an interactive operations guide for cloud
administrators and tenants, for Claude Code, Codex and Cursor.
Built with [trilio-agent-plugin-kit](https://github.com/trilioData/trilio-agent-plugin-kit) 0.1.0.

## What it does

- **Cloud Admins:** Deploy TSR (Kolla Docker, Juju Charms, or OpenShift),
  configure `protector.conf`, register primary and secondary sites, create
  tenant mappings, monitor replication health and storage arrays.
- **Tenants:** Create protection groups, add VMs with boot order, execute
  planned/unplanned failover, failback, non-disruptive test failover, monitor
  RPO compliance, and troubleshoot common errors.

Every answer is grounded in the current published documentation: the skill
fetches the relevant page from the official TSR for OpenStack docs before
guiding you, so guidance stays accurate as the product evolves.

## Install

The plugin installs from the
[Trilio agent marketplace](https://github.com/trilioData/trilio-agent-marketplace).

**Claude Code**

```sh
claude plugin marketplace add trilioData/trilio-agent-marketplace
claude plugin install site-recovery-openstack@trilio
```

Then invoke the skill with `/site-recovery-openstack:tsr-guide`. It asks whether
you are a cloud administrator or a tenant, then walks you through the relevant
workflows.

**Codex**

```sh
codex plugin marketplace add trilioData/trilio-agent-marketplace
codex plugin add site-recovery-openstack@trilio
```

**Cursor**

```sh
cursor-agent plugin marketplace add https://github.com/trilioData/trilio-agent-marketplace
```

Then install the plugin with `/plugins` in a Cursor session.

The plugin connects to the official docs MCP server,
`https://docs-tsr.trilio.io/api/mcp/trilio-site-recovery-for-openstack`. It
needs no token or other setup.

## What you need

- **An agent tool:** Claude Code, Codex or Cursor
- **An OpenStack environment** (≥ Victoria) with Trilio Site Recovery deployed,
  or access to one you are setting up
- **The `openstack dr` CLI plugin**, installed separately with
  `pip install -e ./client` from the TSR repository root

## Covered workflows

### Cloud Administrator

| Workflow | CLI commands |
|----------|-------------|
| Deploy TSR | `deploy/kolla/install.sh`, `deploy/charms/deploy.sh`, `deploy/openshift/scripts/install.sh` |
| Initialize DB | `protector-manage db sync` |
| Register sites | `openstack dr site create`, `openstack dr site list` |
| Tenant mappings | `openstack dr tenant mapping create`, `validate`, `list` |
| Monitor health | `openstack dr replication health`, `openstack dr recovery point list` |
| Usage reports | `openstack dr usage summary`, `openstack dr usage report` |

### Tenant

| Workflow | CLI commands |
|----------|-------------|
| Create PG | `openstack dr protection group create` |
| Add VMs | `openstack dr protection group member add --boot-order N` |
| Planned failover | `openstack dr failover <pg-id> --failover-type planned` |
| Unplanned failover | `openstack dr failover <pg-id> --failover-type unplanned` |
| Failback | `openstack dr failback <pg-id>` |
| Test failover | `openstack dr test failover <pg-id> --auto-network` |
| Cleanup test | `openstack dr test failover cleanup <pg-id>` |
| Monitor RPO | `openstack dr replication health <pg-id>` |
| Recovery points | `openstack dr recovery point list/create <pg-id>` |
| Reconcile | `openstack dr protection group reconcile <pg-id>` |

## Layout

| Path | What it is |
| --- | --- |
| `plugins/site-recovery-openstack/plugin.yaml` | Portable source: manifest and MCP servers. **Edit this.** |
| `plugins/site-recovery-openstack/skills/` | Skills, shared by every agent tool |
| `plugins/site-recovery-openstack/.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.mcp.json`, `mcp.json` | Generated. CI fails if edited by hand |
| `evals/` | Eval cases |

## Develop

```sh
git clone git@github.com:trilioData/trilio-agent-plugin-kit.git .pluginkit -b 0.1.0
pip install pyyaml
python .pluginkit/pluginkit.py validate .
python .pluginkit/pluginkit.py generate .        # after editing plugin.yaml or skills
claude --plugin-dir plugins/site-recovery-openstack      # try it in Claude Code
```

## Release

1. Bump `version` in `plugin.yaml`, run `generate`, and merge.
2. Tag the merged commit `site-recovery-openstack/<version>` and push the tag. The release
   workflow checks the tag against `plugin.yaml`, then builds the zip and GitHub release.
3. The platform group pins the tag in
   [trilio-agent-marketplace](https://github.com/trilioData/trilio-agent-marketplace).

CI needs the org secret `PLUGINKIT_TOKEN` (read access to the kit repo).

## License

MIT. See [LICENSE](./LICENSE).
