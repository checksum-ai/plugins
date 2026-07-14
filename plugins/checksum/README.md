# Checksum for Claude Code

Generate and heal end-to-end tests with [Checksum AI](https://checksum.ai) without leaving your editor. This plugin connects Claude Code to Checksum's hosted MCP server, so you can kick off a test-generation or healing run in the cloud and have Claude poll it to completion.

No local install, no Playwright engine: work runs in Checksum's cloud and lands as a pull request.

## Install

```shell
/plugin marketplace add checksum-ai/plugins
/plugin install checksum@checksum-ai
```

Then run `/mcp`, pick **checksum**, and sign in through your browser. There's no API key to copy or paste — you approve which Checksum projects Claude may act on, and you can revoke it any time from the **MCP connections** card on **My Profile** in the web app.

## Tools

| Tool | What it does |
|------|--------------|
| `checksum_whoami` | Confirms the connection and lists the projects you're authorized for. |
| `checksum_test_generate` | Starts a test-generation run — for a pull request, or for a flow you describe. Returns a `batchId`. |
| `checksum_test_heal` | Starts an auto-heal run for the failing tests in a test run. Returns a `batchId`. |
| `checksum_session_status` | Polls a `batchId`, reporting the agent's progress, file changes, and the `prUrl` it opened. |
| `checksum_test_run_list` | Lists recent test runs, newest first — how Claude finds a failing run without you hunting for an id. |
| `checksum_test_run_download` | Gets a run's results inline, plus download links for the report, traces, screenshots and video. |
| `checksum_session_list` | Lists recent agent sessions. |
| `checksum_session_prompt` | Sends a follow-up instruction into a running session, to steer it mid-run. |

## Use it

Run a command, or just ask in natural language:

```
/checksum:generate    # generate tests for your current branch's changes
/checksum:heal        # heal a failing test run
```

- *"Generate Checksum tests for my changes on this branch."*
- *"Use Checksum to generate a test for the login flow with an invalid password."*
- *"My last Checksum run has failures — heal them."*

**Generating** runs in Checksum's cloud, which checks out your code repository at a branch — so `/checksum:generate` first makes sure your branch is **pushed** (pushing it if needed; never `main`/`master`, never force). Checksum opens a pull request with the tests when the branch has one, so the command resolves the PR for you.

**Healing** works from a test run that already finished, so there's nothing to push — `/checksum:heal` finds a run with failures, shows you why it failed, and opens a PR with the fixes. If a failure turns out to be a real application bug rather than a broken test, it says so instead of forcing the test green.

## Links

- [MCP Server docs](https://checksum.ai/docs/mcp-server)
- [Coding Agent Integration](https://checksum.ai/docs/coding-agent-integration) — the local slash-command path (also does test **detection**)
