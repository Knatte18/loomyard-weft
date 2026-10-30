# Round 3 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed differently, `_lym/plan/14-pr-review-session.md`: `landingshedAllowedImports` gains `internal/stencil` for the `stencil.Fill` call, beside `internal/parentdirective` and `internal/editdirective`.
  The skills list is `languageconfig.AppendSkills([]string{"scribe:prose", "scribe:conversation"}, Deps.LanguageSkills)` as `internal/mergeresolve/spec.go:107` builds it, and the allowlist also gains `internal/languageconfig`.
  The fix key proposed a plain append that avoids importing `internal/languageconfig`; I did not make that change because a plain append drops the de-duplication `AppendSkills` gives (`internal/languageconfig/config.go:144-161`), and `shuttleengine` does not dedupe `Spec.Skills`, so a language skill that repeats `scribe:prose` or `scribe:conversation` would load twice (self-check, fix-proposal lens).
- **F2** (MEDIUM), fixed as proposed, `_lym/plan/13-pr-review-row-waits.md` and `_lym/plan/00-overview.md`: `wait` takes `entryStatus *pullrequest.Status` and `entryReadAt time.Time`.
  `Call` hands over step 6's `Status` and the injected `Now` at that read, or a nil `entryStatus` when step 6 made no read.
  The poll clock starts at `entryReadAt`, the first wake uses the handed status without a `Status` call, and step 4 counts the entry read as the first poll.
  `TestPRReview_Wait` gains the re-entry row with exactly one `Status` call in the first minute, and the `Continue`-before-`Call` row counts from `Call` onward.
  The overview's `One immediate poll on the continue flag` bullet records that the entry read counts as the wait's first poll.
- **F3** (LOW), fixed differently, `_lym/plan/11-pr-review-row-opens.md`: the Recipes paragraph names both header sentences, quoting `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml`.
  In loom-recipe.yaml, PR-Review leaves the sentence and joins the header's set of producer rows that carry no `on_stuck` and whose Stuck halts the run.
  Friction-Reflect stays alone outside the escalate-to-a-human set.
  In darn-recipe.yaml, PR-Review carries no `on_stuck` because its Stuck is always a Halt, and Friction-Reflect carries none because it never returns Stuck.
  The fix key proposed rewriting only the reason, which could leave PR-Review in the `outside the escalate-to-a-human set` frame.
  `loom-recipe.yaml:19-23` defines that set as the no-`on_stuck` rows whose Stuck halts the run, so PR-Review belongs inside it (self-check, fix-proposal lens).

## Self-check

- fix-proposal, MEDIUM, the F3 loom-recipe rewrite kept PR-Review outside the escalate-to-a-human set: fixed as proposed (card 11 Recipes paragraph).
- fix-proposal, LOW, the F1 plain append drops `AppendSkills` de-duplication: fixed as proposed (card 14 skills and allowlist).
- fix-consistency: no findings.
- fix-claims: no findings.
  Its check that a plain append keeps landingshed off `internal/languageconfig` no longer applies, since the self-check fix replaced the plain append.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
