---
description: Generate Checksum end-to-end tests for the current branch's changes. Ensures the branch is pushed and resolves its pull request, then triggers cloud generation and reports the PR Checksum opens. Use when the user asks to generate Checksum tests, cover a PR, or test a flow they just built.
---

# Generate Checksum tests

Checksum generates tests **in the cloud** by checking out the code repository at a specific branch. It can only see commits that exist on the **remote** — so before triggering generation, you MUST make sure the current branch is pushed. Skipping this is the most common mistake: the agent silently generates against stale code and the new changes are never covered.

## Two modes — pick deliberately

Checksum only opens a pull request with the generated tests when it is given a **full pull-request context**: `prNumber` **and** `repoName` **and** `branch`. All three, or no PR is opened.

| You pass | What happens |
|---|---|
| `prNumber` + `repoName` + `branch` | The agent diffs the PR, covers its changes, and **opens a PR** with the tests. |
| `extraInstructions` only (no PR) | The agent generates tests for the flow you describe. **No PR is opened** — the tests land in the session for review. |

So if the user wants a PR, you must find the pull request number. Never promise a `prUrl` for a description-only run — there won't be one.

## Steps

Run these from the user's **code repository** (not the Checksum test repo).

1. **Identify the repo and branch:**
   - Branch: `git rev-parse --abbrev-ref HEAD`
   - Repo slug: `git remote get-url origin`, then reduce to `<owner>/<repo>` (strip the `git@github.com:` or `https://github.com/` prefix and any trailing `.git`).
   - If `HEAD` is detached, or the branch is `main`/`master`, ask the user which feature branch holds the changes to test before continuing.

2. **Ensure the branch is on the remote with all the work:**
   - If `git status --porcelain` shows uncommitted changes the user wants covered, tell them to commit first — the cloud agent only sees committed, pushed code.
   - Check for an upstream: `git rev-parse --abbrev-ref --symbolic-full-name @{u}` (this fails if there's no upstream).
     - **No upstream** → push it: `git push -u origin <branch>`
     - **Upstream exists but local is ahead** (`git rev-list --count @{u}..HEAD` > 0) → `git push`
   - Never force-push. Never push local commits to `main`/`master` on the user's behalf.
   - Tell the user which branch you pushed (one line).

3. **Resolve the pull request for this branch** — this is what makes Checksum open a PR with the tests:
   - `gh pr view --json number --jq .number` (run in the repo, on the branch).
   - **Found a PR** → use that `prNumber`.
   - **No PR yet** → tell the user plainly: *"This branch has no pull request, so Checksum won't open one with the tests. Open a PR first, or I can generate tests for a flow you describe instead."* Then either wait for them to open one, or proceed as a description-only run and **do not promise a PR**.

4. **Trigger generation** by calling the `checksum_test_generate` tool:
   - For a PR-based run, pass **all three**: `prNumber`, `repoName`, `branch`.
   - `extraInstructions`: anything the user said about what to cover (optional alongside a PR; **required** for a description-only run, otherwise the agent gets an empty request).

   It returns a `batchId` and a `sessionUrl`.

5. **Give the user the `sessionUrl` right away** so they can watch the run in the Checksum web app while it works.

6. **Poll** `checksum_session_status` with that `batchId` until `allTerminal` is true. While polling, it also returns the agent's latest messages and file changes, so you can tell the user what it's doing rather than just "still running".

7. **Report the result:**
   - PR-based run → the session's `prUrl`, the pull request Checksum opened with the generated tests, plus the `sessionUrl`. Give both as clickable links, never as raw ids.
   - Description-only run → the `sessionUrl` and what the agent produced. There is no `prUrl`; do not wait for one.
