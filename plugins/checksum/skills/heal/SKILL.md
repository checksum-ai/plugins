---
description: Heal failing Checksum tests for the current branch. Ensures the branch is pushed to the remote, then triggers a cloud heal for a failing test run and reports the PR. Use when the user asks to heal or fix failing Checksum tests.
---

# Heal Checksum tests

Healing runs **in the cloud** against the code repository at a branch, so — exactly like generation — the branch must be pushed to the remote first. Otherwise the heal agent works against stale code and the fix targets the wrong thing.

## Steps

Run these from the user's **code repository**.

1. **Get the failing test run.** Healing operates on a Checksum test run that has failures. If the user didn't give one, ask for the `testRunId` (from the web app or a CI run).

2. **Identify repo + branch and ensure it's pushed:**
   - Branch: `git rev-parse --abbrev-ref HEAD`
   - Repo slug: `git remote get-url origin` → `<owner>/<repo>` (strip the `git@github.com:` / `https://github.com/` prefix and trailing `.git`).
   - If there's no upstream (`git rev-parse --abbrev-ref --symbolic-full-name @{u}` fails) → `git push -u origin <branch>`. If the upstream exists but the local branch is ahead → `git push`.
   - Never force-push, and never push local commits to `main`/`master`. Tell the user which branch you pushed.

3. **Trigger heal** by calling the `checksum_heal` tool with:
   - `testRunId`: from step 1
   - `repoName`: the `<owner>/<repo>` slug
   - `branch`: the current branch — so the heal PR targets your branch and the agent works against your code

   It returns a `batchId`.

4. **Poll** `checksum_status` with that `batchId` until `allTerminal` is true, then report each session's `prUrl` — the pull request Checksum opened with the healed tests.
