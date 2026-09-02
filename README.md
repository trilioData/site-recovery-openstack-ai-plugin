# Trilio Site Recovery for OpenStack — Claude Code Skill

A [Claude Code](https://claude.com/claude-code) skill that serves as an
interactive operations guide for
[Trilio Site Recovery for OpenStack](https://trilio.io). It walks cloud
administrators through deployment, site registration, and monitoring, and
walks tenants through protection groups, failover/failback, test failover, and
RPO compliance — grounded in live product documentation via
[git2docs](https://git2docs.com).

## What it does

- **Cloud Admins:** Deploy TSR (Kolla Docker, Juju Charms, or OpenShift),
  configure `protector.conf`, register primary and secondary sites, create
  tenant mappings, monitor replication health and storage arrays.
- **Tenants:** Create protection groups, add VMs with boot order, execute
  planned/unplanned failover, failback, non-disruptive test failover, monitor
  RPO compliance, and troubleshoot common errors.

Every answer is grounded in the current published documentation — the skill
fetches the relevant doc page from git2docs before guiding you, so guidance
stays accurate as the product evolves.

## Install

In Claude Code, add this repo as a plugin marketplace and install the skill:

```
/plugin marketplace add trilioData/tsr-openstack-skill
/plugin install tsr-openstack-skill
```

Or, to use it without the plugin system, copy the skill into your skills
directory:

```bash
git clone https://github.com/trilioData/tsr-openstack-skill /tmp/tsr-skill
cp -r /tmp/tsr-skill/skills/tsr-guide ~/.claude/skills/tsr-guide
```

## Setup

The skill pulls live documentation from git2docs. To connect:

1. In [git2docs](https://git2docs.com), go to **Settings → Access tokens** and
   create a token (`g2d_…`, shown once). Export it:

   ```bash
   export GIT2DOCS_TOKEN=g2d_…
   ```

2. Add the MCP server to your project's `.mcp.json`
   (see [`.mcp.json.example`](./.mcp.json.example)):

   ```json
   {
     "mcpServers": {
       "trilio-site-recovery": {
         "type": "http",
         "url": "https://git2docs.com/api/mcp/YOUR_ORG/trilio-site-recovery-for-openstack/agent",
         "headers": {
           "Authorization": "Bearer ${GIT2DOCS_TOKEN}"
         }
       }
     }
   }
   ```

   Replace `YOUR_ORG` with your git2docs organisation slug.

3. In Claude Code, invoke the skill:

   ```
   /tsr-guide
   ```

   It will ask whether you are a cloud administrator or tenant, then walk you
   through the relevant workflows.

## What you need

- **Claude Code** — the CLI, desktop app, or web app
- **An OpenStack environment** (≥ Victoria) with Trilio Site Recovery deployed,
  or access to one you are setting up
- **The `openstack dr` CLI plugin** — installed separately with
  `pip install -e ./client` from the TSR repository root
- *(Optional)* A git2docs access token if you want the skill to pull live
  documentation (without it, the skill still works using its built-in
  reference material)

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

## License

MIT — see [LICENSE](./LICENSE).
