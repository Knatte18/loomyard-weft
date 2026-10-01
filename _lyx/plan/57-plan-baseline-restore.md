# Card 57 — plan-baseline-restore

**Create:**
- `internal/websterengine/planbaseline.go`
- `internal/websterengine/planbaseline_test.go`
- `internal/webstercli/restoreplan.go`
- `internal/webstercli/restoreplan_test.go`

**Edit:**
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/validate.go`
- `internal/webstercli/cli.go`
- `cmd/lyx/helptree_test.go`
- `internal/websterengine/fingerprint_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/webstercli/verbs_test.go`

**Uses:**
- `internal/websterengine/state.go`
- `internal/webstercli/acceptaudit.go`
- `internal/webstercli/sync.go`

**Intent:**
First half of PR #321 round-4 finding D's third item: the way forward named for a suspect plan path is `git checkout <head> -- <path>`, which fails because `_lyx/plan` is not tracked on the task branch.
A standalone run's plan is in no git repository at all, so no git command can name a working restore either.
Webster therefore keeps the content behind every plan hash it records, and a new verb writes it back.

`planbaseline.go`:

- The baseline store is `<WebsterDir>/plan-baseline/`, one file per recorded content named by its hex SHA-256.
  It lives under `WebsterDir` so the fabric sync carries it with `state.json` and a cross-machine resume finds it.
- `storePlanBaseline(websterDir, planDir string, hashes map[string]string) error` writes each plan file's bytes to its hash's file when that file is absent, through a temp file and rename.
  It is content-addressed, so a repeat write is a no-op and no stored copy is ever wrong.
  An empty `websterDir` is a wiring error, returned rather than resolved against the working directory.
- `ErrPlanBaselineMissing`, an exported sentinel.
- `RestorePlan(st *State, geom Geometry) (restored []string, err error)` restores every plan file `changedPlanFiles` reports: a recorded file is written back from the store after checking the stored bytes hash to the recorded value, and a file the run never recorded is removed.
  It checks every needed copy before writing anything, and refuses with `ErrPlanBaselineMissing` naming each file without one, changing nothing, with `way forward: reset the branch to the run's start commit with git and run "lyx webster run --fresh"`.
  A state with no `PlanFileHashes` refuses the same way.
  It returns the restored file names sorted and never touches `state.json`.

`fingerprint.go`: `restampFingerprint(st *State, planDir, websterDir string) error` and `RestampPlanBaseline(st *State, planDir, websterDir string) error` call `storePlanBaseline` after computing the hashes and before mutating `st`, so a store failure leaves the state's hashes unchanged.
`restampAndSaveFingerprint` passes `geom.WebsterDir`.
Every caller passes its geometry's `WebsterDir`: `beginbatch.go`, both `recordbatch.go` sites, `rebaseline.go` (`deps.Geom`, added by card 55) and `internal/webstercli/validate.go` (`c.geom.WebsterDir`).
`runlevel.go`'s two state-initialisation sites call `storePlanBaseline` with the hashes they record before saving the new state.

`internal/webstercli/restoreplan.go` builds `lyx webster restore-plan` on `acceptAuditCmd`'s shape: `clihelp.ShouldAbort`, the state-mutation lease, `LoadState`, no run refusing as `accept-audit` does, `websterengine.RestorePlan`, lease released, then `fabricSync` labelled `restore-plan` with a sync failure reported with `fabricSyncWayForward`.
The success envelope carries `restored`, the file names written back or removed, empty when nothing differed.
`Short`: `restore every plan file that differs from the plan the run recorded`.
`Long` says it writes back each plan file from the copy webster kept when it recorded the plan's hashes, removes a plan file the run never recorded, never changes `state.json` or the task worktree, refuses changing nothing when a copy is missing, and gives the example `lyx webster restore-plan`.
`cli.go` registers it and lists it in the parent's help text;
`cmd/lyx/helptree_test.go`'s webster entry lists `restore-plan`.

`doc.go` gains a paragraph on the plan baseline store and `restore-plan`.

Tests:

- `planbaseline_test.go`:
  - `TestStorePlanBaseline_ContentAddressed`: two files stored → one file per hash holding the bytes, and a second call rewrites nothing (modification time unchanged).
  - `TestRestorePlan_RestoresEditedAndRemovesUnrecorded`: after a restamp, one card edited, one card deleted and one new card added → all three reported, the edited and deleted cards hold their recorded bytes and the new card is gone.
  - `TestRestorePlan_RefusesMissingCopy`: a recorded hash with no stored copy → `ErrPlanBaselineMissing` naming the file, and the edited card on disk is unchanged.
  - `TestRestorePlan_NothingChanged`: an unchanged plan → an empty list.
- `fingerprint_test.go`: the existing `restampFingerprint` call passes a temp webster dir, and a restamp leaves one stored copy per recorded hash.
- `rebaseline_test.go`'s `RebaselineDeps` literals set `Geom` from the fixture's geometry, and `verbs_test.go`'s `RestampPlanBaseline` call passes `fx.CLI.geom.WebsterDir`.
- `restoreplan_test.go` (integration-tagged like `acceptaudit_test.go`): an edited card → exit 0, `restored` names it, and the card file holds the recorded bytes with `state.json` byte-identical;
  no `state.json` → the error envelope names `lyx webster run`.

**ImpactSummary:** Every plan-hash restamp and run initialisation also stores the hashed content under `_lyx/webster/plan-baseline`, and a new `lyx webster restore-plan` verb writes a changed plan back from it; the restamp functions gain a webster-dir argument.
