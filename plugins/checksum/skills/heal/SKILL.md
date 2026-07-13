---
description: Heal failing Checksum tests for the current branch. Ensures the branch is pushed to the remote, then triggers a cloud heal for a failing test run and reports the PR. Use when the user asks to heal or fix failing Checksum tests.
---

# Heal Checksum tests

Healing runs **in the cloud** against the code repository at a branch, so — exactly like generation — the branch must be pushed to the remote first. Otherwise the heal agent works against stale code and the fix targets the wrong thing.

## Steps

Run these from the user's **code repository**.

1. **Get the failing test run.** Healing operates on a Checksum test run that has failures.
   - If the user didn't name one, call `checksum_test_run_list` (newest first, `failedCount` per run) and offer them the most recent run with failures instead of making them go hunt for an id.
   - To see *why* a run failed before healing it, call `checksum_test_run_download` with the `testRunId` — it returns the per-test results inline, plus signed URLs for the report, trace, screenshots and video.

2. **Identify repo + branch and ensure it's pushed:**
   - Branch: `git rev-parse --abbrev-ref HEAD`
   - Repo slug: `git remote get-url origin` → `<owner>/<repo>` (strip the `git@github.com:` / `https://github.com/` prefix and trailing `.git`).
   - If there's no upstream (`git rev-parse --abbrev-ref --symbolic-full-name @{u}` fails) → `git push -u origin <branch>`. If the upstream exists but the local branch is ahead → `git push`.
   - Never force-push, and never push local commits to `main`/`master`. Tell the user which branch you pushed.

3. **Trigger heal** by calling the `checksum_test_heal` tool with:
   - `testRunId`: from step 1
   - `repoName`: the `<owner>/<repo>` slug
   - `branch`: the current branch — so the heal PR targets your branch and the agent works against your code

   It returns a `batchId`, a `testRunUrl`, and one `sessionUrl` per failing test (healing fans out one session per test).

4. **Give the user the `testRunUrl` and `sessionUrls` right away** so they can watch the healing in the Checksum web app.

5. **Poll** `checksum_session_status` with that `batchId` until `allTerminal` is true. It also returns the agent's latest messages and file changes, so you can report what it actually changed.

6. **Report the result:** each session's `prUrl` — the pull request Checksum opened with the healed tests — plus the links above. Give them as clickable links, never as raw ids.

## Important

Healing can legitimately conclude a failure is a **real application bug**, not a broken test. When `checksum_session_status` reports a heal outcome saying so, tell the user plainly — do not "fix" the test to make a genuine bug go green.
