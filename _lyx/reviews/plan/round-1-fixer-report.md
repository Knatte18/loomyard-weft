# Fixer report — plan, round 1

## Changes

- **F1** (MEDIUM, fixed as proposed): card 2's **Edit:** adds `//docs/overview.md` after `//internal/orchengine/doc.go`.
  The Intent gains a sentence saying the orch module line's verb list `lyx orch start|status|refresh|distill|stop|cleanup` gains `rules`.
  The ImpactSummary now names the orch module line of `docs/overview.md` among the places that gain the verb.
- **F2** (MEDIUM, fixed as proposed): in card 2, `writeHubRules` step 2 now says what happens when the `Push` error wraps `gitrepo.ErrPullRebaseFailed`.
  The refusal then names the board's diverged recovery: run `git pull --rebase` in the board dir (`rulesPath` with `rulesRel()` trimmed from its end), resolve the conflict, run `lyx board sync`, then re-run `lyx orch rules --from <file>`.
  Step 4 refers to the same recovery.
  Every other push error keeps the re-run wording.
  **Uses:** gains `internal/gitrepo#ErrPullRebaseFailed`.
  `TestWriteHubRules` gains the first-push row: no write call, the file unchanged, and the refusal names `git pull --rebase` and `lyx board sync`.
- **F3** (LOW, fixed as proposed): in card 1, the both-present row of `TestRenderFiles_WriteWithoutLeadingComment` keeps `want: []string{"## Commands\n", testIndex, "Decision for <slug>:"}` beside its `wantOrder`.
- **F4** (LOW, fixed as proposed): card 2's `fakeRulesBolt` gains a seedable `upstreamMoved` flag.
  When `upstreamMoved` is set and `unpushed` is above 0, `PullThenCommitWritten` returns a `diverged` skip without running `write`.
  The re-run-after-failed-push row seeds `upstreamMoved`.
  The row's closing clause now says which assertion catches each dropped push.
  A dropped first push fails that row on its exit code and fields, as well as on its call log.
  A dropped second push fails the new-file row on the fake's `unpushed` 0, and the call log of every row that commits.

## Self-check

- fix-claims: no findings.
- fix-proposal, LOW: step 4's rebase-conflict branch had no `TestWriteHubRules` row.
  Fixed as proposed: a row whose last push wraps `gitrepo.ErrPullRebaseFailed` keeps the new content with `committed: true`, `unpushed: true` and the fake's `unpushed` 1, and the refusal names `git pull --rebase` and `lyx board sync`.
- fix-consistency, MEDIUM: the same missing step-4 row.
  This duplicates the fix-proposal finding, and the same row fixes it, including the fake's `unpushed` 1 that this fork asked for.
- fix-consistency, LOW: D3 still names "the next run of the verb" for a failed push, with no exception for a conflicting rebase.
  Fixed as proposed: card 2's step 4 gains a sentence saying the conflict wording in steps 2 and 4 is the plan's call on the case D3 leaves open.
- fix-affected-set: no findings, and no finding was left unchecked.

## Disputed

None.

## Deferred

None.
