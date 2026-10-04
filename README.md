# Sanction

Request one-off human approval before an agent acts. Sanction evaluates spend, tools, credentials and provisioning, then approves, escalates to a human, or denies the request.

This MIT package contains a hosted MCP connection and four skills for Cursor and Grok Build. It is prepared for marketplace review; publication and authenticated host testing are pending. Grok Build is distinct from consumer Grok and Grok Bot; this package does not establish listings or compatibility on those surfaces.

## Set up a Sanction agent

Complete the [wallet claim and agent setup](https://github.com/ericlovold/sanction/blob/main/docs/QUICKSTART.md#1-create-a-wallet) first. Owner-email verification revokes pre-claim agent keys. Use a dedicated `pxy_…` agent key, never a management key.

The package sends authenticated MCP requests only to `https://getsanction.com/mcp` (Streamable HTTP). Cursor sends the agent key in `x-api-key`; Grok Build sends it as a bearer token. This is the ten-tool wallet profile, including execution mandates and credential injection. The package has no hooks, helper scripts, bundled executables, or telemetry.

### Cursor

Copy this repository into `~/.cursor/plugins/local/sanction` without overwriting an existing plugin, then run Developer: Reload Window. Set `SANCTION_AGENT_KEY` in the plugin's Configure control; the manifest declares the variable but contains no value. Confirm the four skills and the `sanction` MCP server in Customize. An installed marketplace plugin with the same name takes precedence. Team policy may restrict local imports. [Cursor local testing](https://cursor.com/docs/plugins), [variables](https://cursor.com/docs/reference/plugins#variables)

### Grok Build

Supply `SANCTION_AGENT_KEY` to the Grok process environment through your normal secret-management workflow before launching Grok. Do not put a literal key in shell history, a committed file, a URL, or chat. The `.mcp.json` uses Grok's native `bearer_token_env_var`; Cursor's Configure setting does not populate Grok's environment. If the variable is absent, the server receives no bearer header and authentication may fail or prompt; stop setup and supply the dedicated key.

From the parent of a local clone, install the reviewed package:

```bash
grok plugin install ./sanction-plugin --trust
grok plugin details sanction
```

Start a new session or reload plugins, and confirm the plugin is enabled and its four skills and MCP server are present. Trust activates plugin components. Organization policy may block installation or MCP access. [Official Grok Build plugin guide](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-pager/docs/user-guide/09-plugins.md)

### First connection check

In either host, inspect the ten available tools and call only `sanction_wallet_status` with `{}`. Confirm a normal response without displaying the agent key. `sanction_authorize` is not a dry run. A missing tool inventory or authentication error is a setup failure.

## Ask for one human approval

After the read-only connection check, ask for a synthetic one-off decision:

> Call `sanction_authorize_tool` for `demo.noop` with server `demo`, arguments
> `{"message":"My first approval"}`, `require_approval: true`, and
> `approval_reason: "I want to try a one-off approval."` Make one request,
> show its request ID, then stop. Do not execute, retry, or poll automatically.

No policy edit is needed; hard denials still apply. After the owner decides, ask
the host to check that request once. On `next_action: retry_with_grant`, retry the
same authorization once with identical fields plus the returned `grant_id`.
Only `authorized: true` with `next_action: proceed` permits that attempt. This
synthetic test still executes nothing.

The skills read the machine decision, pause pending requests for human input,
and stop on denial, reused authority, errors or unknown outcomes. They do not
start fresh requests to evade those results. Approval binds the exact tool,
server and arguments; authorization and grant redemption do not execute the
target. Authenticated Cursor and Grok Build tests of this lifecycle remain pending.

## What the MCP exposes

Same ten tools as the hosted wallet. Do not invent others.

| Tool | What it does |
|------|----------------|
| `sanction_authorize` | Ask before purchase, subscribe, transfer, or API credit top-up. |
| `sanction_authorize_tool` | Ask before another MCP tool, shell, deploy, or email send. |
| `sanction_authorize_capability` | Ask before acquiring a new skill, plugin, integration, or API. |
| `sanction_authorize_provision` | Ask before provisioning seats, licenses, or infrastructure (resource + dollars). |
| `sanction_check_authorization` | Check a pending request once; return a usable one-use grant after approval. |
| `sanction_wallet_status` | Today / MTD spend and token totals, plus pending approvals. |
| `sanction_request_execution` | Mint a short-lived mandate (JWT) for a child agent or counterparty. |
| `sanction_inject_credential` | Retrieve a vaulted secret under that mandate (audit-logged). |
| `sanction_log_tokens` | Record LLM token usage against the token budget. |
| `sanction_log_outcome` | Record a confirmed business outcome (feeds cost-per-outcome ceilings). |

## Skills included

| Skill | When to use |
|-------|-------------|
| `before-spend` | Before purchase, subscribe, transfer, or API credit top-up. |
| `before-tool` | Before another tool or external action, including explicit one-off human approval. |
| `wallet-status` | Start of long or expensive work, or after a budget error. |
| `handle-escalation` | When authorization returns `next_action: wait`; check once, then pause for the human. |

v1 is MCP + skills only (portable Agent Orchestration connector). No rules, agents, commands, or hooks.

## Honest limits

This plugin wires the **cooperative** hosted wallet. The host must ask before acting. It does not intercept every MCP `tools/call`. Skipping the ask is not a bypass the engine can see.

For intercepted `tools/call`, register the upstream and point the host at the broker (`/mcp/broker/<name>`), not this plugin's wallet URL. See [The agent wallet](https://github.com/ericlovold/sanction/blob/main/docs/AGENT-WALLET.md).

## License

MIT for this standalone package, extracted from Sanction’s MIT `cursor-plugin/` directory. The Sanction product is [FSL-1.1-MIT](https://github.com/ericlovold/sanction/blob/main/LICENSE).

## Packaging and submission

The hosts share `skills/` and `assets/` but use separate manifests and MCP configuration:

| Host | Manifest | MCP config | Authentication source |
|------|----------|------------|-----------------------|
| Cursor | `.cursor-plugin/plugin.json` | `mcp.json` | Cursor plugin variable `SANCTION_AGENT_KEY` |
| Grok Build | `.grok-plugin/plugin.json` | `.mcp.json` | Grok process environment `SANCTION_AGENT_KEY` |

Cursor submissions use [Marketplace publish](https://cursor.com/marketplace/publish) and require a public repository and local testing. Grok Build submissions use a PR to [xai-org/plugin-marketplace](https://github.com/xai-org/plugin-marketplace/blob/main/CONTRIBUTING.md), with a public source commit pinned by full SHA, generated component index, and passing catalog validation. A submission is not acceptance.

### Host acceptance checklist (not yet completed)

Record the host version, four discovered skills, ten tools, a successful wallet-status call, and the synthetic request/pause/check/exact-redemption lifecycle above. Test denial and stop behavior without executing the target. No authenticated host result is claimed by this package.
