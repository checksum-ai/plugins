---
description: Generate Checksum end-to-end tests for the current branch's changes. Ensures the branch is pushed to the remote, then triggers cloud generation against that branch and reports the PR. Use when the user asks to generate Checksum tests, cover a PR, or test a flow they just built.
---

# Generate Checksum tests

Checksum generates tests **in the cloud** by checking out the code repository at a specific branch. It can only see commits that exist on the **remote** — so before triggering generation, you MUST make sure the current branch is pushed. Skipping this is the most common mistake: the agent silently generates against stale code and the new changes are never covered.

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

3. **Trigger generation** by calling the `checksum_generate` tool with:
   - `repoName`: the `<owner>/<repo>` slug from step 1
   - `branch`: the current branch from step 1 — this is what the cloud agent checks out
   - `prNumber`: include it if the user referenced a specific PR
   - `extraInstructions`: anything the user said about what to cover

   It returns a `batchId`.

4. **Poll** `checksum_status` with that `batchId` until `allTerminal` is true, then report each session's `prUrl` — the pull request Checksum opened with the generated tests.
