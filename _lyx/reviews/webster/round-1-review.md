---
verdict: APPROVED
---

# Round 1 review

Range: `status.json` gives `product.parent` = `main` and no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which is `b7d4fd0ba..HEAD`, one commit per plan card (1 to 14).

I found no defect.
`lyx gate test` over `verifytree`, `loomshed`, `loomcli`, `landingshed`, `pairteardown`, `darnengine`, `loomrecipe`, `shedrecipe`, `gitrepo`, `fabricengine` and `cmd/lyx` passes, which includes the test-budget, config-strictness and stencil-claim guards.

## Focus questions settled

- **Card 6 and cards 11/12, one verify gate in two forms.**
  `loomshed.NewVerifyGate` in `internal/loomshed/verifygate.go` keeps the round form when `mergeBase` is nil: same empty-command pass with a warning, same `impactset.Derive` base, same lint, same `./...` fallback for the `tmux` join.
  A told merge base selects the whole-diff form: the lint base is `mergeBase()`, the command runs in full with `BaseCommand` equal to it, an empty command and a read error from either source are returned errors.
  `wire` fills `Env.VerifyCommand` from `planVerifySource` and leaves `VerifyMergeBase` nil for loom; only `wireDarn` sets `darnMergeBase`.
  The shed-recipe spec's gate vocabulary names `verify` and both forms.
- **Card 12, recipe read once.**
  `armAt` calls `resolveRecipe` right after `resolveRunID`, which stores the recipe on `loomCLI.recipe`.
  `recipeRouting`, `newShed`, `verifySource`, the `wire` branch and `reworkRowFor(c.recipe)` all read that field.
  No other `ReadSeed` call in the touched loomcli files reads a recipe; `seedAndCommitBootstrap` preserves the recorded one through `resolveSeedRecipe`.
  `reject` is a lightweight verb and still passes through `armAt`, so `c.recipe` is set when `rejectCmd` runs.
- **Cards 2 and 3.**
  Finalize calls `commitParentRecords` after the catch-up merge-in and before `parentOpener`, and again on the merge-in-required retry; the Stuck reason matches the card and a nil seam is skipped.
  `RemovePair` reads `landed` before removal only while the task worktree is present, ORs it with `req.Landed`, and settles only after a successful removal; `settleClaim` implements the card's rule table, including the `claimHeld` warn-only path and the read-failure no-write path.
  Batten's `teardownRequest` sets `Landed: true`.
- **Card 1.**
  `verify-<n>.log` is numbered one above the highest existing log, `pruneLogs` keeps the newest `verifyLogKeep` plus the one the failure record names, and a prune failure is logged only.
  The landing gate's `recordFailure` sets `LogPath` from `result.Log`, and `checkedPublishFailure` shape-checks it with `IsLogPath`.
  Webster's gate reads the log of the result it reports on.
- **Cards 8 and 11.**
  `NewDarn` refuses a budget below 1 as its first act after the status-path coherence check, with the card's message, and `applyVerifyAttempts` errors on a missing row or entry.
  `internal/darnengine` imports no resolver and is told every path.
- **Per-card mechanical checks.**
  The only `Delete` group in the plan is card 1's `logName` and `publishFailureLogName`; a grep finds no remaining reference to either, nor to `PublishFailureLog` or `paths.Log`.
  The plan has no `Rename` group.
- **Test economy.**
  I found no new test that duplicates an existing one, and every behavior the diff adds has a test: the new loomcli helpers, `darnWrite.Call`, the `DarnWrite` entry, `NewDarn`, `DarnSpec`, `MergeBase`, `DirtyPaths`, the parent-records commit and the claim settlement.

No `## Focus departures`.
