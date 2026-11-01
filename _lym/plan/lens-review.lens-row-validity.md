```yaml
lens: row-validity
outcome: reported
```

Row-validity lens: per-row reachability, whether each row fails when the code is wrong, and new tests that repeat existing ones, across cards 1–14.

**MEDIUM — Card 11, D4 "a beam the local Bolt still names as this machine's but origin names as another's gets its runs held": the planned `TestAdopt` row cannot reach that state.**
`world.deps().ReadOwners` returns `w.owners` on every call (`internal/adoptcli/adopt_test.go:104`). `Reconcile` only sets `w.reconciled` and returns `w.report` (`adopt_test.go:105-108`), so the owners read before the reconcile and after it are always identical. The card lists new `world` state only for tasks, pause updates, holds files and tries files. It needs a hook on `Reconcile` that rewrites `w.owners`, like the existing `race` hook on `SetOwners` (`adopt_test.go:63`, `:139-141`). Without it, the row runs with one owner reading and would pass even if the code read the owners before the reconcile.

**MEDIUM — Card 11, D4 "written before the owner write": the order assertion has nothing to compare against.**
`world.order` records only resumes, child resumes, beam-ups and suite starts (`adopt_test.go:154`, `:174`, `:190`, `:198`). The `SetOwners` fake records `w.ownerReq` but appends nothing to `order` (`adopt_test.go:120-143`). The card names no `order` entry for the owner write or for the pause and holds writes. So a hold written after the owner write would not fail the row.

**LOW — Card 2, `TestPlanPass` row "a held run recorded owned elsewhere on an uncovered probe tick gets none": the input cannot occur.**
`Held` is set only when a live entry of the run's own scope carries `HoldCaller`, and that entry also sets `Covered` (`internal/pauserecord/record.go` `CoverageOf`). So `Held` with `covered` false is impossible. The current code already returns none for `OwnedElsewhere` on a covered probe tick (`internal/dobbyengine/passes.go` `planPass`: `trigger.Lift || (trigger.ProbeTick && !covered)`). The row that would fail if the new early return were missing is a held, owned-elsewhere run with a `Lift` trigger.

**LOW — Card 6, `TestStartReady_FrontTask` row "a front task of another project holds nothing and does not start": the start half fails unless the row sets the task's `Parent`.**
`readyTask` sets `Parent: "prime"` (`internal/dobbyengine/starts_test.go:72-75`), and the harness worktree is `prime` with an empty slug (`passes_test.go:142`). A released front task whose `Project` is a grouping project under the prime therefore passes `readyToStart` and starts through the ordinary loop. The row must give the task a `Parent` other than `prime`, and the card does not say so.

**LOW — Card 5, `TestSettle`: the claim that `OpenFront` and `FrontOf` "are reached through these rows" is half false.**
In card 5's Intent, `Settle` uses only open-front detection (records an open front entry with no record) and the gone/done check of recorded entries. `FrontOf` is used only by cards 12 and 13. No card-5 row reaches `FrontOf`, so its "not done, whatever its status" rule is first exercised at card 12.

**LOW — Card 3, test economy: `TestFrontTask` and `stepFrontTask` test the same refusals.**
`TestFrontTask` checks that `UpsertTask`, `UpsertTasksBatch` and `MergeTasks` refuse a changed `front`/`on_land` and a project/kind change, and accept the stored values sent back. `stepFrontTask` drives exactly those behaviours through `lym board upsert`, `upsert-batch` and `merge`, which the decision record's `PATTERN-refusal-way-forward` constraint requires.
- The engine-level refusal rows repeat the CLI step. `TestFrontTask` should keep only what the CLI cannot reach: `FileFrontTask`'s draft-then-release, and its refusal of a payload carrying `front`.
- The `TestRename_Refusals` front row repeats `stepFrontTask`'s `lym board rename` row.
- "A non-front entry's project change is judged as before" is already covered by `TestProjectRules` (`internal/boardengine/project_test.go:48`).

**LOW — Card 3, the `TestRename_Refusals` front row needs helpers the card does not list.**
Rows that seed a special entry build their board with `newProjectBoard` and `seedBoardJSON` (`internal/boardengine/rename_test.go:168-171`, "BeamProject" row). The card's Uses lists only `newRenameBoard`. A released front entry has to be seeded the same way, since a board write cannot set `front`.

**LOW — Card 10, `TestAdopt` row "a start that fails is the subject's warning naming `lym adopt <subject>`": the fake cannot fail one subject only.**
`world.suiteErr` is one error for every start (`adopt_test.go:56`, `:197-200`). Showing that a failed start is that subject's warning, with the other subject's check still started, needs an error per subject. The card does not list that `world` field.

Tool calls:
1. Read the `internal/dobbyengine` test function list and `TestOwnedElsewhere`, to check the Card 2 and Card 6 rows against their harness.
2. Read `passFake`, `newPassHarness` and the start of `TestStartReady_*` (`passes_test.go`, `starts_test.go`), to check fake fields, the harness geometry and `readyTask`.
3. Read the rows of `TestCancelRun`, the head of `TestRename_Refusals` and the pause-record setup of `TestDobbyRuns_PairAtItsOwnAnchor`, to check the Card 4, 3 and 2 rows. None of these test bodies were in context.
