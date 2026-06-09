# Checksum for Claude Code

Generate and heal end-to-end tests with [Checksum AI](https://checksum.ai) without leaving your editor. This plugin connects Claude Code to Checksum's hosted MCP server, so you can kick off a test-generation or healing run in the cloud and have Claude poll it to completion — the same engine behind the GitHub `/checksum generate` command.

No local install, no Playwright engine: work runs in Checksum's cloud and lands as a pull request.

## Install

```shell
/plugin marketplace add checksum-ai/plugins
/plugin install checksum@checksum-ai
```

On enable, Claude Code prompts for your **Checksum API key** (grab it from the web app at [app.checksum.ai](https://app.checksum.ai) — the same key used by the CLI and CI). It's stored in your system keychain, never in a file.

## Tools

| Tool | What it does |
|------|--------------|
| `checksum_generate` | Start a test-generation run for a pull request, or for a flow you describe in plain text. Returns a `batchId`. |
| `checksum_heal` | Start an auto-heal run for the failing tests in a test run. Returns a `batchId`. |
| `checksum_status` | Poll a `batchId` until the run finishes, then read each session's `prUrl` — the pull request opened with the new or healed tests. |

## Use it

Just ask Claude in natural language — it picks the right tool:

- *"Use Checksum to generate tests for PR 142 in acme/web on branch feature/checkout."*
- *"Use Checksum to generate a test for the login flow with an invalid password."*
- *"Heal the failing tests in Checksum test run 7f3a… and open a PR with the fixes."*

Claude calls `checksum_generate` / `checksum_heal`, gets a `batchId`, polls `checksum_status` until it's done, and reports the pull request URL(s).

## Links

- [MCP Server docs](https://checksum.ai/docs/mcp-server)
- [Coding Agent Integration](https://checksum.ai/docs/coding-agent-integration) — the local slash-command path (also does test **detection**)
