---
description: Heal failing Checksum tests. Finds a test run with failures, inspects why it failed, then triggers a cloud heal and reports the PR with the fixes. Use when the user asks to heal or fix failing Checksum tests.
---

# Heal Checksum tests

Healing operates on a **Checksum test run that already finished with failures**. It works from that run — not from your local working tree — so there is nothing to push and no branch to line up first. All you need is the `testRunId`.

Checksum opens **one** heal session covering all the failing tests in the run (not one per test) and, by default, a pull request with the fixes.

## Steps

1. **Find the failing test run.**
   - If the user named one, use it.
   - Otherwise call `checksum_test_run_list` — it returns recent runs newest-first with a `failedCount` and a `testRunUrl` each. Offer the most recent run that has failures rather than making the user hunt for an id.

2. **Understand the failure before healing it** (do this unless the user just wants it fixed blind):
   - Call `checksum_test_run_download` with the `testRunId`. That returns the per-test results inline (which tests failed, and their error messages) plus a signed link to the HTML report.
   - To dig into one specific test, call it again with that test's `testId` (from the returned `tests[]`) to get its trace, screenshots and video.
   - Tell the user briefly what actually failed. A failure can be a genuine application bug, not a broken test — see below.

3. **Trigger the heal** by calling `checksum_test_heal` with:
   - `testRunId`: from step 1. **This is the only argument you need.**
   - Do **not** pass `branch` unless the user explicitly asks the heal PR to target a different branch. It defaults to the branch the test run executed on, which is almost always what you want — passing your local code branch here will target a branch that may not even exist in the tests repo.
   - `autoCreatePR`: leave it alone (defaults to opening a PR with the fixes).

   It returns a `batchId`, a `testRunUrl`, and a `sessionUrl`.

4. **Give the user the `testRunUrl` and `sessionUrl` right away** so they can watch the healing in the Checksum web app.

5. **Poll** `checksum_session_status` with that `batchId` until `allTerminal` is true. It also returns the agent's latest messages and file changes, so you can report what it actually changed.

6. **Report the result:** the session's `prUrl` — the pull request Checksum opened with the healed tests — plus the links above. Give them as clickable links, never as raw ids.

## Important

Healing can legitimately conclude a failure is a **real application bug**, not a broken test. When the heal outcome says so, tell the user plainly — do not "fix" the test to make a genuine bug go green. That is the single most valuable thing this tool tells you.
