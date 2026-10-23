# Card 40 — idle-release-keeps-finalize

**Create:**
- `plan:internal/loomcli#finalizeKeepsLock` -> `func finalizeKeepsLock(statusPath, statusLockPath, verifyDir string) func() bool`
- `plan:internal/loomcli#TestFinalizeKeepsLock` -> `func TestFinalizeKeepsLock(t *testing.T)`

**Edit:**
- `internal/loomcli#loopIdleRelease` -> `func loopIdleRelease(locks *parentlock.Store, self func() parentlock.Owner, agents func() ([]parentlock.AgentState, error), bound time.Duration, covered func() bool, keep func() bool) func()`
- `internal/loomcli#idleReleaseOn` -> `func idleReleaseOn(locks *parentlock.Store, self func() parentlock.Owner, agents func() ([]parentlock.AgentState, error), bound time.Duration, awake func() time.Duration, covered func() bool, keep func() bool) func()`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#TestLoopIdleRelease`
- `internal/loomcli#TestIdleReleaseLogLevels`
- `internal/battenshed#ChildParentLock`
- `internal/battenshed#innerRunProducer.loopLockStep`
- `internal/battenshed#TestInnerRun_ParentLockReleasesAStalledLoopsRecord`
- `internal/battencli#childParentLock`
- `internal/battencli#TestChildParentLock`
- `internal/landingshed/template.yaml`
- `internal/landingshed#Config`

**Uses:**
- `internal/verifytree#ReadMarker`

**Extends:**
- `internal/battenshed/doc.go`

**Intent:** No idle release frees the parent lock while its owner's run is at Finalize and running, or while a verify marker for the run is live (D10), so a Finalize `landing` gate run longer than `parent_lock_idle_release_min`, with the fixer session waiting through it, keeps the lock.
The check stands in both idle-release paths, beside each one's idle rule.
In the loop's wake hook, `loopIdleRelease` and `idleReleaseOn` take a `keep` check and release nothing, restarting the watch, while it answers true; `specFor` tells `finalizeKeepsLock`, which reads the run's status file under its lock (current producer Finalize and state running) and the verify directory's running marker through `verifytree.ReadMarker` (a live marker of any site).
In batten's backstop, `ChildParentLock` gains `KeepsLock func() (bool, error)`, and `loopLockStep` releases nothing while it answers true; a read error keeps the lock and warns once per episode.
battencli's `childParentLock` fills it over the task worktree's status and verify directory.
The owner-pause arm of `lockStep` and `ownedElsewhere` keep releasing unless `keepsLockForFinalize` holds, and `FinalizePastMerge` is unchanged: they act only while an owner pause or another machine's ownership stands, and Finalize's pre-landing re-check catches any such release.
`parent_lock_idle_release_min`'s template comment and `Config` field doc drop their PR-Gate clause and name the Finalize exception (D13).

`TestLoopIdleRelease` gains rows: a run at Finalize and running with agents waiting past the bound keeps its record, and a live verify marker keeps it; `TestFinalizeKeepsLock` pins the check over a status file and a marker; `TestInnerRun_ParentLockReleasesAStalledLoopsRecord` gains rows for batten's loop arm keeping a stalled-looking record while `KeepsLock` answers true, releasing it once it answers false, and keeping it with one Warn when `KeepsLock` fails.

**ImpactSummary:** Both idle-release paths gain one keep check; their callers are loom's `specFor` and battencli's lock seams, and the owner-pause and other-machine arms are unchanged.
