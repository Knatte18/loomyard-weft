# Round 1 fixer report — plan review

`lyx loom validate-plan` exits clean on the edited plan (only the informational `create-new-unit` findings).
`approved:` in `00-overview.md` is untouched and still `false`.
No git command was run, and nothing outside the plan directory and the two review files was written, except the optional friction note.

## Fixed

- **F1 (BLOCKING, test budget):** `//cmd/lyx/testdata/test-budget.yaml` joins the `Edit` list of Cards 1 and 2.
  Each card's `Intent` raises the `internal/fabricengine` row to the package's top-level test count after the card, and each `ImpactSummary` names the file.
  `00-overview.md` gains a Shared Decisions bullet stating the rule for any card that adds top-level tests.
- **F3 (Card 7 footprint):** `spawn.go` and `mutation.go` join Card 7's `Edit` list, and its `Intent` names the two `recordPushIfAdvanced` calls and the `Kind` and `refDetail` docs.
- **F4 (Card 2, existing tests):** Card 2's Coverage now says what happens to each test that pins the old behaviour.
  `TestAddRollback_WarpBranchLeftBehindUnderEmptyPrefix` flips to deletion and is renamed.
  `TestAddRollback_RefusedWarpBranchDeletionLogsWarn` is reworked into the lease-refusal case with the WARN assertion.
  `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites` is deleted, since its purpose is gone and its remaining behaviour is covered by the `SkipPush` case.
  The stale empty-prefix comments in `destructivegaps_integration_test.go` and `livestate_verbs_test.go` are rewritten with the flips.
  The new file no longer repeats the local-deletion and moved-on-origin assertions.
- **F5 (Card 2, `SkipPush` row):** the row is now a step-11 runner that pushes for real and then reports an error under `SkipPush`, which deletes only the local branch and leaves origin's.
- **F6 (Card 3 coverage):** a task-pair `sync` case (two `push_spawned` entries) joins the prime case, and the `Intent` states that `lyx config`'s post-edit `sync` is the same in-process verb, so those tests pin it.
- **F7 (Cards 4 and 5, created files):** `add_rollback_branch_integration_test.go` and `add_pushretry_integration_test.go` join both `Edit` lists, and both `Intent`s say so.
- **F8 (Card 10):** the "exempt identifier alone passes" row is dropped, and the "non-exempt identifier fails" row is renamed to a side-named identifier case; the Coverage line follows.
- **F9 (missing constraints):** a Shared Decisions bullet names `PATTERN-comment-line-breaks`, `PATTERN-markdown-link-integrity` and `PATTERN-no-denied-recovery`.

## Deferred, with reason

- **F2 (Cards 4 and 5 typed `Edit` instead of `Rename`):** not applied.
  The fix was tried and `lyx loom validate-plan` rejected it with blocking `containment-file-overlap` findings: Card 5's member glyphs overlap the whole-file `Edit` targets of Cards 1, 2, 6, 7 and 8 (`destroy.go`, `fabric.go`, `weftwiring.go`), and the same holds for Card 4's `hub.go` against Card 5.
  Making it pass would mean retyping the file targets of every other card to member glyphs, a rewrite of the plan's footprint that no finding licenses.
  Cards 4 and 5 are restored to their `Edit` form (with the F7 additions), and a Shared Decisions bullet in `00-overview.md` states why the symbol renames stay `Edit` over files, so the next reviewer does not re-raise it.
  The enumerated file lists stay, with the staleness risk the review named; the module-wide AST rename and the build catch a missed file.
