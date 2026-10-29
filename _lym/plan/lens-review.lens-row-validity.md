```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. Scope: the plan's test rows, whether they can reach and fail, and whether they repeat existing tests.

**Findings**

1. **BLOCKING** — Card 5 (shuttle-unawaited-session), D7 "Done, Stuck, error and pause stop it". An unawaited run has no output files, so `StopStrand` never records it stopped, and no row checks this.
   `stopRecorded` writes `stopped` only when `stopOutcome(runOutcomeRunning, rs.OutputFiles) == runOutcomeStopped`. A record whose output files "all exist" keeps `running`, and an empty list counts as all existing (`internal/shuttleengine/stop.go:68-84`, `stop.go:18`, `Run.Stop` at `stop.go:35`).
   So the session's run record still reads running after every stop the row makes. That is the stale running record `readAgentActivity` reads.
   `TestRunner_StartSession` only asserts that a ready start writes a record reading running. Add a row in which `StopStrand` on an unawaited run records it stopped, and handle an empty `OutputFiles` list in `stopOutcome`.

2. **MEDIUM** — Card 13 (pr-review-row-waits), D8 "a dirty tree waits" and D9 "merged ... on a dirty tree halts naming the uncommitted files". The row's tree read has no seam a test can script.
   `newVerifyGate` hard-wires `dirty: verifytree.DirtyPaths`, which runs real git on `WorktreeRoot` (`internal/landingshed/verifygate.go:128-142`).
   Card 13 adds `CommitsMissing`, `MailAgent`, `PauseCovered`, `Now` and `Sleep`, but no dirty-paths seam. It also says the gate's dirty seam "is the row's tree read" without naming that seam.
   So these `TestPRReview_Wait` rows cannot script a dirty tree: dirty tree waits, head moved during the gate discards the verdict, merged on a dirty tree halts, and continue with a dirty tree keeps waiting. Add a `DirtyPaths func() ([]string, error)` seam on `PullRequestDeps`, give the gate the same seam, and list it on the card.

3. **MEDIUM** — Card 13. The new seams have no stated nil meaning, but hand-built receivers drive `Call` into the wait.
   Card 11's wiring guard covers only `ParentMode`, `EntryField`, `Origin`, `API`, `PushTask` and `RecordPath`. Card 13 makes every yes path enter `wait`, which calls `Now`, `Sleep`, `PauseCovered`, `CommitsMissing` and `MailAgent`.
   Card 13 says only that `TestPRReview_Call`'s yes rows "script a Status that ends the wait". Those rows, and envkit-built rows, would hit nil funcs.
   State each seam's nil meaning, or extend the fail-loud guard to them and list the guard row.

4. **MEDIUM** — Card 14 (pr-review-session). The same gap applies to `Sessions`, `SessionLive` and `RegisterSession`.
   `ensureSession` runs on every yes entry, before `wait`. The existing `TestPRReview_Call` and `TestPRReview_Wait` rows build `PullRequestDeps` without these seams, and the card neither states a nil meaning nor names those rows as gaining the fakes.

5. **LOW** — Card 14. `TestPRReview_Session` needs the `landing-template-pr-review` stencil on disk under `Deps.StencilsDir`, but the card lists no seeding helper.
   `internal/testkit/stencilkit` exists for this and is missing from `**Uses:**`. The other landingshed tests that read stencils go through `StencilsDir` fixtures, for example `internal/landingshed/describe_test.go`.

6. **LOW** — Card 2 (beam-pull-request-key). Two `TestLoadConfig` rows cannot be written in the table as it stands, and one repeats existing coverage.
   - The table writes only the board template and an anchor worktree layer (`internal/dobbyengine/config_test.go:28-50`, `writeBeamLayer`).
   - "`required` and `off` from the repo default" needs a board-layer override field the table lacks.
   - "the key in the local layer is refused" needs a writer for `LayerLocal`, the untracked file under the anchor's `.lym` (`internal/configengine/layers.go:40`). No helper for it is listed.
   - The local-layer refusal also repeats what card 2's own `TestModules_EveryTemplateKeyClassified` `wantLocalDenied` row pins, and configengine's mechanism is already tested.
   Drop the local-layer row, or list the helper.

7. **LOW** — Card 11 (pr-review-row-opens). `TestPRReviewEntry`'s "`pr_review_setting` is refused as unknown" row repeats "an unknown key is refused naming it". Both reach the same `configRejectUnknown(cfg, "gate_key")` branch.

8. **LOW** — Card 15 (finalize-through-pull-request). The "a pending refusal then a success merges once retried" row needs `fakePullRequestAPI.Merge` to return a sequence of results. Card 11 defines the fake only as "records calls, scripted results", and card 15 adds no per-call result list for `Merge`.

9. **LOW** — Card 16 (teardown-closes-pull-request). `TestCloseOpenPullRequest`'s rows assert warnings through the process-global logger, but the card does not say the test stays serial.
   `teardown_test.go` is tier0 and its tests may run with `t.Parallel`. The card does not state the logcapture use or that the test stays serial, as `TestPRReview_Call` does.

10. **LOW** — Card 13. The row "a `Deps.Lock` store holds no record for the parent after every row" passes whatever `PRReview` does, because `PRReview` holds no guard and never reads `Deps.Lock`.
    It would fail only if code took the lock through that exact store. As written it gives the "takes no parent lock" criterion almost no test.

**Tool calls I made**
- `internal/dobbyengine/config_test.go` `TestLoadConfig`, the header lines of four test files, and `verifyGate` with `check`: needed the dobby table's layer fields, the test files' tiers, and the gate's struct and dirty seam.
- `TestSpec_Validate`, `TestStartup_HandleReturned` and pairteardown's `fakeSubstrates`: needed the fixtures that card 5's and card 16's rows run on.
- `configengine.Dirs` and its layers, `newVerifyGate`, fixreview stencil seeding, the fake engine's `PrepareCalls` and the `runlock_test` names: needed to check that card 2's, card 13's and card 5's rows can reach their branches.
- The `LayerLocal` definition, stencil-reading tests and `stencilkit`: needed to confirm the local-layer location and the stencil seeding helper card 14 lacks.
