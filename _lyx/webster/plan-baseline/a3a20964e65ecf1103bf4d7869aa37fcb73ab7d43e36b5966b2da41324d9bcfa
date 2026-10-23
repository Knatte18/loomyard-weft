# Card 19 — batten-cancel-verb

**Create:**
- `plan:internal/battencli#battenCLI.cancelCmd` -> `func (c *battenCLI) cancelCmd() *cobra.Command`
- `plan:internal/battencli#hubCancelDeps` -> `func hubCancelDeps(location *lyxcwd.Location, slug string) (cancelDeps, error)`
- `plan:internal/battencli#slugPauseRefs` -> `func slugPauseRefs(record pauserecord.Record, slug string) []tenderengine.EntryRef`
- `plan:internal/battencli#hubPauseLive` -> `func hubPauseLive(record pauserecord.Record) bool`
- `plan:internal/battencli#clearRunClaim` -> `func clearRunClaim(read func() (status *string, found bool, err error), write func(status *string) error) (cleared, found bool, err error)`
- `plan:internal/battencli#stopRecordedTackler` -> `func stopRecordedTackler(markerPath string, stop func(guid string) error) (string, error)`
- `plan:internal/battencli#takeBattenLocks` -> `func takeBattenLocks(driverLockPath, runLockPath string, wait time.Duration) (release func(), busy string, err error)`
- `plan:internal/battencli#TestTakeBattenLocks` -> `func TestTakeBattenLocks(t *testing.T)`
- `plan:internal/battencli#TestCancelPauseEntries` -> `func TestCancelPauseEntries(t *testing.T)`
- `plan:internal/battencli#TestClearRunClaim` -> `func TestClearRunClaim(t *testing.T)`
- `plan:internal/battencli#TestStopRecordedTackler` -> `func TestStopRecordedTackler(t *testing.T)`
- `plan:internal/battencli#TestCancelCmd_RefusesBeforeAnything` -> `func TestCancelCmd_RefusesBeforeAnything(t *testing.T)`
- `plan:internal/battencli#TestCancelIntegration_BlockedRunWithNoPair` -> `func TestCancelIntegration_BlockedRunWithNoPair(t *testing.T)`
- `plan:internal/battencli#TestCancelIntegration_KeptPairJudgeOnly` -> `func TestCancelIntegration_KeptPairJudgeOnly(t *testing.T)`

**Edit:**
- `internal/battencli#Command`
- `internal/battencli#ResumeInWindow`
- `internal/battencli#battenCLI.resolvePersistentPreRun`
- `internal/battencli#reconcileRun` -> `func reconcileRun(location *lyxcwd.Location, slug string, judgeOnly bool) (fabricengine.OriginReport, error)`
- `internal/battencli#TestCommand_EveryVerbArity`

**Uses:**
- `plan:internal/battencli#cancelRun`
- `plan:internal/battencli#childReachedParent`
- `internal/battencli#removeRunRecords`
- `internal/battencli#takeCancelMarker`
- `internal/battencli#readCancelMarker`
- `internal/battencli#refuseSelfAddress`
- `internal/battencli#spawnStatusPush`
- `internal/battencli#battenProgram`
- `internal/battencli#wireForHub`
- `internal/battencli#seedBoardTask`
- `internal/battencli#newOverviewSource`
- `internal/pairteardown#Teardown.EndForCancel`
- `internal/pairteardown#CancelLockWait`
- `internal/proc#CreationTime`

**Intent:**
Add the verb `lyx batten cancel <slug> [--remove]` (D1) and wire `cancelRun`'s production seams.

`cancelCmd` builds the verb with operator audience, `cobra.MaximumNArgs(1)` and the `--remove` flag; its `Long` says what the verb ends, removes and clears, that the pair is kept by default and `--remove` removes it with a snapshot and archives and deletes its task branch, which refusals it makes, that origin unreachable ends only, and its JSON envelope keys.
Its `RunE` applies `refuseSelfAddress` to the positional first, so an omitted or reserved slug refuses before any path is resolved, then resolves the cwd and location and applies `RefuseUnlessPrime`, so a pair agent cannot reach it, then `hubCancelDeps` and `cancelRun`, and prints the report as the envelope: `slug`, `state`, `row`, `ended`, `strands_stopped`, `tackler_stopped`, `pair_removed`, `snapshot_tag`, `archive_tags`, `records_removed`, `claim_cleared`, `lifted`, `push_spawned`, `notice_posted`, `end_only` and `way_forward`.
What the verb can skip: every wait a teardown makes for a working driver, since it ends the run mid-turn; what bounds it: the prime, the one slug named and D2's refusals.
What `--remove` lets through: a forced removal that discards the pair's uncommitted paths, and the deletion of an unlanded task branch, locally and on origin.
What bounds it (D5): the local snapshot tag is taken first, and a refused snapshot removes nothing; every deleted tip is held by an archive tag on the warp origin, and a failed tag push deletes no branch; and D2's reached-parent refusal keeps it from ever removing a pair whose run here has work that reached the parent.
`Command` adds it beside `cleanup`, and the group's `Long` and example list it; `resolvePersistentPreRun` skips `cancel`, which arms no shed spec, as it skips `cleanup`.
The package doc atop `cli.go` gains the verb's paragraph: the order D3 gives, the marker, the kept pair's rerun step.

`hubCancelDeps` wires each seam over the hub:
`HubPaused` reads the pause record (`hubgeom.PauseGeometry`, `pauserecord.Read`) through `hubPauseLive`, which is true only for a live hub-scope entry, so a slug entry of the cancelled run, or of another run, never refuses the cancel (D2, D8); `ExemptGate` is `pauserecord.SetGateExemptAll`;
`taskWorktreePresent`; `reconcileRun(location, slug, true)`; `pullBolt` and `boardengine.ReadOwners` with `owners.Effective` and `machinename.Name`; batten's status and records directory through `shedrun`; the board entry through `boardengine.OpenHub`;
`ChildReachedParent` through `childReachedParent` over batten's status, `readChildStatus` and `battenshed.FinalizePastMerge` with the restepped and parent-holds closures the wire's Inner-Run seams use;
`readCancelMarker` and `takeCancelMarker` bounded by `pairteardown.CancelLockWait`; `runtime.GOOS`, `runproc.List` and `proc.CreationTime`; a topology's `PairParts`, `TaskBranchOnOrigin` and `DiscardTaskBranch`; reed's session listing for the pair's session; `pairteardown.New(location)`'s `EndForCancel` and `RemovePair`; `TakeBattenLocks` through `takeBattenLocks(shedrun.DriverLock(location, slug), shedrun.RunLock(location, slug), pairteardown.CancelLockWait)`;
`takeBattenLocks` takes each lock through `lock.AcquireWriteLockWithin` bounded by `wait`, the driver lock first, and a busy run lock releases the driver lock already taken before the busy path is returned, so a busy report holds no lock and the phase's retry takes neither lock against this process's own handle;
`StopTackler` through `stopRecordedTackler(shedrun.TacklerMarker(location, slug), runner.StopStrand)`, with the runner a shuttle runner over the prime's reed geometry; `stopRecordedTackler` reads the marker with `sheddriver.ReadTacklerMarker`, returns no guid for an absent marker, and otherwise stops the strand it names and returns its guid or the stop's error;
`removeRunRecords` under `lock.AcquireWriteLock` of the status lock with `fabricCleanupDeps`;
`ClearClaim` through `clearRunClaim` over the hub board's `GetTask` and `SetStatus(slug, …)`: it clears only a status `boardengine.IsRunStatus` reads as run-form, leaves a done status, a non-run status and an absent entry alone (D7), and reports whether it cleared and whether an entry was found;
`tendercli.HubDeps` and `tenderengine.Lift` over `slugPauseRefs`; `spawnStatusPush`; and `orchcli.NotifyPrime(location, battenProgram, line)`.
`slugPauseRefs` returns the ref of each live entry whose scope is exactly the slug, never a hub entry or another slug's (D8).
`reconcileRun` takes `judgeOnly`, set on its `ReconcileScope`; the closures in `Command` and `ResumeInWindow` pass false, so a resume moves refs as before.

`TestCommand_EveryVerbArity` gains the `cancel` row.
`TestCancelCmd_RefusesBeforeAnything` runs the verb through `RunCLIIn` from a temporary directory outside any repository: an omitted slug and a reserved slug each refuse on the envelope with `refuseSelfAddress`'s message, and a named slug there refuses with the location's resolve error; the prime refusal itself is `RefuseUnlessPrime`'s, pinned by `TestRefuseNonPrime`.
`TestCancelPauseEntries` is a table over pause records: `slugPauseRefs` picks the slug's own live entries and not a hub entry, another slug's entry or a lifted one, and `hubPauseLive` reads true for a live hub entry and false for a slug entry of the cancelled slug, another slug's entry and a lifted hub entry.
`TestClearRunClaim` is a table over fake reads and writes: a run-form status is cleared to nil; a done status, a non-run status and an absent entry write nothing; a read error and a write error are returned.
`TestStopRecordedTackler` is a table over a temporary marker path: no marker stops nothing and returns no guid; a marker stops its `strand_guid` and returns it; a stop error is returned.
`TestTakeBattenLocks` is an untagged test over temporary lock paths with a wait of a few milliseconds, never zero, since `AcquireWriteLockWithin` with a zero wait reports busy before its first try: both locks free are held until `release` is called, so a try-acquire of each on a fresh handle reads not acquired, and free after it; the run lock held on another handle reports busy naming the run lock's path, and a try-acquire of the driver lock on a fresh handle then succeeds; the driver lock held on another handle reports busy naming the driver lock's path, and a try-acquire of the run lock on a fresh handle then succeeds, since it was never taken.
`TestCancelIntegration_BlockedRunWithNoPair` is a tier-1 integration test over a hubforge hub (the first acceptance criterion): a board task claimed `running · Worktree-Create`, a batten seed, a blocked status at Worktree-Create with no pair, and a tackler marker naming a guid no strand holds, run through the verb: the envelope reports the records removed and committed, the tackler's guid stopped and the claim cleared, the records run directory and the scratch's tackler marker are gone, the board entry has no status, the overview source built with `newOverviewSource` over the prime lists no line for the slug, and a second cancel refuses as nothing left to cancel.
`TestCancelIntegration_KeptPairJudgeOnly` is a tier-1 integration test over a hubforge hub with a pair added for the slug, a batten seed and a paused status at Run-Shed, and the pair's records branch on origin advanced past the local one: the cancel without `--remove` keeps the pair, leaves the local records branch at its old tip, so the reconcile it ran moved no ref, clears the claim and reports the kept pair's way forward naming `lyx batten cancel <slug> --remove`.

**ImpactSummary:** `reconcileRun`'s two callers, both in `cli.go`, pass false and keep today's moves; the verb tree gains one sibling, and `lyx help index` picks it up from the binary.
