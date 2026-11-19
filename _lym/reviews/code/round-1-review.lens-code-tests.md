```yaml
lens: code-tests
outcome: reported
```

Reviewing the tests of the diff (code-tests lens): for each behaviour the diff adds or changes, which test row would fail if the behaviour broke.

**Behaviours and the tests that pin them**
1. `watchRows` refuses a name that is no row, and the refusal names the key, the row, the recipe, `way forward` and the recipe's rows: `TestWatchRows_UnknownRowNamesTheRecipeRows`. Pinned.
2. `watchRows` wraps only the named rows: the `UnwatchedRow` step of `TestWatchRows_TellsParentOncePerVisit`. Pinned.
3. One notice per visit:
   - a call with an unchanged history sends no notice: the `CallAfterAYield` step;
   - a new verdict opens a new visit: the `CallAfterANewVerdict` step;
   - a folded repeat opens a new visit: the `CallAfterAFoldedRepeat` step.
   All in `TestWatchRows_TellsParentOncePerVisit`. Pinned.
4. The wrapped producer calls the inner producer once per call: the `*calls` check in the same test. Pinned.
5. The notice text opens `[row reached] the run reached &lt;row&gt;`: the closing prefix check in the same test, but it checks only `lines[0]`. Partly pinned.
6. A failed post leaves the row's outcome and error as they are: `TestWatchRows_FailedNoticeLeavesTheCall`. Pinned.
7. The visit is recorded before the mail goes out, so a failed post is never retried: none.
8. The notice goes out before the inner call: none.
9. A failed or missing status read, or a failed record read or write, only warns and the row's call still runs: none.
10. The wrapper passes the inner producer's `OutputPointer` and non-nil error through unchanged: none. The stub always returns an empty pointer and a nil error.
11. `newShed` wraps only when `NotifyParentOn` is non-empty, passes `c.recipe`, `RowNoticeRecord` and `c.env.NotifyParent`, and returns the refusal: none. The reach report says no mapped test reaches `loomCLI.newShed`.
12. `shedrun.RowNoticeRecord` resolves to `row-notice.json` under `ScratchDir`: none.
13. `ReadRowNotice` reads an absent record as the zero value, and `WriteRowNotice` creates the directory: covered indirectly by `TestWatchRows_TellsParentOncePerVisit`, because the record lives in a `scratch/` subdirectory the fixture never creates.
14. The `notify_parent_on` key loads from `loom.yaml` and from `run_settings` into `Config.NotifyParentOn`: none for the value. `TestModules_EveryTemplateKeyClassified` covers only the classification.
15. `newStrandRig` sets a hold notifier that posts through `orchcli.NotifyAsker`, signed `GATE`: none. The reach report says no mapped test reaches `gatecli.newStrandRig`.
16. Mutation survivors:
   - `internal/loomcli/rownotice.go:84` negates `if err := p.notify(...); err != nil`, which changes only a log warning. No reasonable test kills it, so it is no finding.
   - `internal/loomcli/rownotice.go:92` changes the `1 +` constant. Any positive constant keeps `verdictCount` strictly growing with each verdict, so no behaviour-level test can kill it. No finding.
17. Stencil and doc prose: exempt.

**Findings**

- **MEDIUM**, design. `internal/loomcli/arm.go:315` (`newShed`).
  No test reaches the wiring that turns the `notify_parent_on` config into wrapped producers. That covers the empty-list short cut, the record path and the parent poster passed in, and the refusal reaching the run's steps. A wrong `paths` value or poster, or a dropped `watchRows` call, would pass every test.
  Evidence: the reach report shows no mapped test reaches `internal/loomcli.loomCLI.newShed`; the code is at `arm.go:315-328`.

- **LOW**, design. `internal/loomcli/rownotice.go:79-81`, in `tell`.
  The documented contract "The visit is recorded before the mail goes out, so a failed post is never retried" has no test. `TestWatchRows_FailedNoticeLeavesTheCall` calls the wrapper only once. A second call with the same history, asserting no second post, would pin it.
  Evidence: the doc comment at `rownotice.go:62` and the single `Call` at `rownotice_test.go:110`.

- **LOW**, design. `internal/loomcli/rownotice.go:57-60`, in `Call`.
  The promised order, notice before the row's call, is not pinned. Neither is the pass-through of the inner producer's `OutputPointer` and error. A swapped order, or a wrapper that dropped the pointer or error, would pass. The stub `countingProducer` returns an empty pointer and a nil error, and no test checks the notify order against the inner call.
  Evidence: `rownotice_test.go:25-28`; the order is stated in the `loomcli` package doc ("before the row's call") and the summary.

- **LOW**, design. `internal/loomcli/rownotice.go:66-69`, in `tell`.
  The warn-only path for a missing or unreadable status file is untested. A regression that panicked, or that blocked the inner call when the status is absent, would pass.
  Evidence: every test writes the status before calling (`rownotice_test.go:84` and `rownotice_test.go:109`).

- **LOW**, design. `internal/shedrun/rownotice.go:24` (`RowNoticeRecord`).
  The new path accessor has no row in the shedrun path-accessor table, where its sibling `WakeRecord` has one.
  Evidence: `internal/shedrun/paths_test.go:96` holds `WakeRecord`; grep finds no `RowNoticeRecord` in shedrun's tests.

- **LOW**, design. `internal/loomengine/config.go:566` (`NotifyParentOn`).
  No test pins that a `notify_parent_on` list value decodes into the field. Each sibling parent key has an "overridden" row in `TestLoadConfig_Loads`, such as "Discussion-Review parent-review keys overridden" at `config_test.go:233-236` and the code-review keys at `config_test.go:239-240`, and none exists for this key.
  Evidence: `config_test.go:94-146` and `config_test.go:205-240`; grep finds no `NotifyParentOn` or `notify_parent_on` in `internal/loomengine/*_test.go`.

- **LOW**, design. `internal/gatecli/cardfixer.go:135-138` (`newStrandRig`).
  No test covers the new hold notifier. A rig built without it, or one signing with the wrong program or posting through the wrong route, passes every test. The comparable runners' notifiers may have no tests either, which limits how far this can be raised.
  Evidence: the reach report says no mapped test reaches `internal/gatecli.newStrandRig`.

- **NIT**, consistency. `internal/loomcli/rownotice_test.go:96-98`.
  The failure message says "want each to open", but the check reads only `lines[0]`. The notices sent after the first are never checked for their row or prefix. Loop over `lines`, or reword the message.
  Evidence: `rownotice_test.go:96-98`.

**Tool calls**
1. Numbered lines 64–95 of `internal/loomcli/rownotice.go`, to place the mutation survivors at lines 84 and 92. My context had the code without line numbers.
2. Grepped the shedrun and loomengine tests for `RowNotice`, `WakeRecord` and the config-key tests, because the manifest did not cover test files.
3. Read `internal/loomengine/config_test.go:100-175` and `:205-236`, the `templateDefaults` table and the override rows, to see how sibling keys are pinned.
4. Read the `templateDefaults` doc and its check, to learn whether the table must be complete. It need not be, so a missing entry there is no finding.
