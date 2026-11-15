```yaml
lens: forced-edits
outcome: reported
```

Task: forced-edits lens review of the 17-card plan.

**Findings**

1. **BLOCKING** (card 15, pr-record-tracked): the new `PullRequestDeps.CommitRecord` field breaks the row's own required-field rule. `internal/landingshed/deps.go:344-345` says the row "treats every field but `BeamRecord` and `DropBeamRecord` as required" and "returns an error naming the first one left unset." Card 15 says a nil `CommitRecord` "commit[s] nothing", but it lists neither the function that checks the fields nor the doc line at deps.go:344. It also leaves out the test fixtures that build `PullRequestDeps` (`newTestDeps` and `NewTestDeps` appear only on card 16). Either the check exempts the new field, or every fixture sets it. As written, existing PR-Review and Finalize tests fail the check.

2. **MEDIUM** (card 13, shared-bolt-pull): moving `pullBolt` into `boardengine` may create an import cycle. Its body calls `fabricengine.NewBolt`, `fabricengine.BoltWrite` and `fabricengine.NewMutations`, so `boardengine` would have to import `fabricengine`. The import scan found `"internal/boardengine"` in `internal/fabricengine/*.go`; I did not tell production files from test files. `internal/fabricengine/doc.go:1311` also says fabric "holds `internal/boardengine`". If production `fabricengine` imports `boardengine`, card 13 does not compile, and card 14 depends on it. The card needs to settle where the function goes, for example `fabricengine` itself or `pairteardown`.

3. **MEDIUM** (card 7, cancel-intent): `lym batten cancel` will keep the batten record, but several texts still say it removes the record, and no card lists them:
   - `internal/boardengine/runlock.go:94`, in `RunLockedError.Error`, tells the operator the cancel "ends its processes, removes its records and clears this status".
   - `docs/overview.md:1597` says "removes the run's records".
   - `internal/battencli/cli.go:449`, the `batten` group's Long text in `Command`, says it "removes its records". Card 8 lists the file `cli.go`, but its Intent changes only the package doc.

4. **LOW** (card 8, cancel-readers-intent): card 8 deletes `shedrun.CancelMarker`, but these comments and probe texts still name the cancel marker and no card edits them:
   - `internal/shedrun/doc.go:51-52` names the deleted `CancelMarker`. Card 8 lists that file under Uses, not Edit.
   - `internal/pairteardown/doc.go:154` ("no cancel marker of the slug stands") is listed under Uses on cards 8 and 9.
   - In `internal/battencli/startguard.go`: the `startGuard.Marker` field doc (31), the `hubStartGuard` doc (83), the `startGuard.refuse` doc (254) and its probe text "the cancel marker" (282), and the `refuseCancelMarker` probe text (463).
   - `internal/battencli/arm.go:759`, the `preDriveCheck` doc.
   - In `internal/dobbyengine`: the `RunView.CancelMarker` doc (`passes.go:39`), `ended.go:33`, `waitreminder.go:66` and `runnotice.go:167`. Card 11 edits `RunView` but not these comments.

5. **LOW** (card 6, board-display-from-fsm): the Intent says "No decision reads the state half", but `internal/boardengine/render.go` `atCell` (around line 950) does. It shows the producer alone only when the state is `running`, so a row now written as `starting · X`, `landing · Finalize` or `halted · X` renders as the whole status. The display change is not acknowledged, and no render test is planned for it.

6. **LOW** (card 6): `TestBattenSeams_*` in `internal/battencli/wire_seams_test.go:697,714` asserts the claim text `running · Worktree-Create`. It stays valid only because card 6 keeps the claim word, so a change of mind there forces this test too.

7. **LOW** (card 1, shed-run-intents): card 1 lists `internal/shedengine/doc.go` under Uses only, while `status.go:13,26` say "six legal State values". Card 1 updates `status.go` but leaves unchecked whether `doc.go` states the closed set too.

Every member glyph I resolved with `lym quarry` came back found, test functions and landingshed members included.

**Tool calls**
1. grep for `pullBolt`/`planPass` callers, `TakeMarker` fakes, cancel-marker and "removes the run's records" wording: callers and texts outside the pre-read.
2. `lym quarry resolve` on the cards' glyphs: output was unreadable and gave no verdict.
3. `lym quarry resolve` on the test and landingshed glyphs, plus a grep for cancel-marker comment sites: verified the glyphs and located the stale comments.
4. Package-import scan, `PullRequestDeps` check lookup, `render.go` `atCell`, tests that assert board status text, `judgeStatus` callers: import cycle, required-field rule and readers of the state half were not in context.
5. `deps.go:340-350` and assertions of the claim text in tests: needed the exact required-field contract.
