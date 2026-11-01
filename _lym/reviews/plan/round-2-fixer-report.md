# Fixer report — plan, round 2

## Changes

- **F1** (MEDIUM, fixed as proposed) — `04-batten-cancel-export.md`: the Create entry `endOnlyError` is replaced by `cancelJudgingEndOnly(ctx, slug, remove, deps cancelDeps) error`.
  `Cancel` builds `hubCancelDeps(prime, slug)` and returns `cancelJudgingEndOnly`, which runs `cancelRun` and turns an `EndOnly` report into the end-only error, nil for a full cancel.
  `TestCancelRun`'s end-only rows assert that error from `cancelJudgingEndOnly` over the row's fake, and its full-cancel rows assert nil.
  The skip sentence now names `Cancel` as its subject, and the ImpactSummary names the new composition.
- **F2** (MEDIUM, fixed as proposed) — `13-check-fix-loop.md`: an Intent sentence drops the failed-tier notice's `fix it, then run <re-run> in the prime` way forward, leaving the tier and the log, with `Failed`'s lines saying what comes next.
  The `TestRunCheck` failed-tier row asserts the notice names neither `in the prime` nor `lym gate test --gate adopt`.
- **F3** (LOW, fixed as proposed) — `12-check-green-end.md`: `docs/overview.md` joins the Edit list after `internal/battenshed/doc.go`, and an Intent sentence has the overview's batten paragraph name the adopt check's green end in place of `lym adopt`, beside `lym beam land`.
- **F4** (MEDIUM, fixed as proposed) — `11-adopt-holds.md`: D2 warns on `rewritten`, `diverged`, or `ahead` with `Moved` unset; the worktree after `lym fabric pull in` is `CodeWorktree` for a records branch and `Worktree` for a code branch; a replayed records branch (`ahead`, `Moved` set) gets no warning.
  `TestAdopt` gains that no-warning row, and its ahead row reads `Moved` unset.

`lym loom validate-plan` reports `ok: true`, with informational `create-new-unit` findings only.

## Self-check

- fix-consistency, LOW: card 12's ImpactSummary leaves out the overview edit — fixed as proposed; it now names the overview's batten paragraph.
- fix-consistency, LOW: card 13's ImpactSummary leaves out the failed-tier notice — fixed as proposed; it now says the notice drops the hand fix in the prime.
- fix-claims: no findings.
- fix-affected-set, LOW: the before-any-tier notice shares the `post` and keeps `in the prime` — fixed as proposed; card 13 has that notice name no worktree, with fixing what stopped the gate then a re-run of `lym adopt <subject>` as its way forward, and its `TestRunCheck` row asserts neither `in the prime` nor `lym gate test --gate adopt`.
- fix-proposal, MEDIUM: a `hubFixDeps` build error after a failed tier leaves the failed end with no way forward — fixed as proposed in its way forward; the build-error line names the error, fixing its cause and a re-run of `lym adopt <subject>`.
  Fixed differently in its test: no `TestRunCheck` or `TestFailedEnd` row, since the branch sits in `hubCheckDeps` over the hub, which neither test's fakes reach; the card states it as a propagated raw read error, exempt as transient, as card 12 does for its `hubGreenDeps` twin.

## Disputed

None.

## Deferred

None.
