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
| `checksum_test_generate` | Start a test-generation run for a pull request, or for a flow you describe in plain text. Returns a `batchId`. |
| `checksum_test_heal` | Start an auto-heal run for the failing tests in a test run. Returns a `batchId`. |
| `checksum_session_status` | Poll a `batchId` until the run finishes, then read each session's `prUrl` — the pull request opened with the new or healed tests. |

## Use it

Run a command, or just ask in natural language:

```
/checksum:generate    # generate tests for your current branch's changes
/checksum:heal        # heal a failing test run
```

- *"Generate Checksum tests for my changes on this branch."*
- *"Use Checksum to generate a test for the login flow with an invalid password."*
- *"Heal the failing tests in Checksum test run 7f3a…"*

Generation and healing run in **Checksum's cloud**, which checks out your code repository at a branch. So the commands first make sure your current branch is **pushed to the remote** (pushing it if needed — never `main`/`master`, never force), then pass that branch to Checksum so the agent sees exactly the code you're working on. Claude then polls until the run finishes and reports the pull request URL(s).

## Links

- [MCP Server docs](https://checksum.ai/docs/mcp-server)
- [Coding Agent Integration](https://checksum.ai/docs/coding-agent-integration) — the local slash-command path (also does test **detection**)
