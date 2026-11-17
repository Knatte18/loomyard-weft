```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points and the record's Added after Discussion entries.

**Decisions, from the board entry's Delivers and Exports lines, plus the Added after Discussion entries**

- **G1: landed pairs and cancelled pairs without keep are torn down by FSM state.** The teardown skips a pair that a pause, a hold or another owner covers. A cancelled run's batten record goes in the same step.
  - Built by C1, C2 and C8.
- **G2: the local pair of a run is removed in local mode.** This covers a run that this machine's hand-over mark names while another machine owns it.
  - Built by C3.
- **G3: kept pairs are listed with their age in `lym beam status`.** One notice per dobby start names `lym fabric remove &lt;slug&gt;`.
  - Built by C4.
- **G4: each pre-upgrade kept-cancel record gets a finished keep cancel intent at the first dobby start.**
  - Built by C5.
- **G5: orphans are noticed and their first sight is recorded.** The record is a machine-local file named in lymdirs. An idle orphan goes after the grace, with its branches archived.
  - Built by C6.
- **G6: `lym tender resume` spawns no pair push for a transition.** It still pays a push that a pause deferred.
  - Built by C7.
- **G7: the hub workspace file is regenerated after each teardown.**
  - Built by C1, at the batch end of `tearDownLeftovers`.
- **Exports: the orphan grace constant and the first-sight path.**
  - Built by C6: `dobbyengine.OrphanGrace` and `fabricengine.OrphanFirstSightPath`.
- **Added (parent): the hub stop writes no per-run pause or cancel intent.**
  - This governs D9, which another part of the Sketch delivers. This part's Delivers line asks nothing of it.
  - No change in the diff touches the hub stop, and none should, so there is no finding.
- **Added (parent): only the printed `parent_beam` names the prime root.**
  - This governs D8, which another part delivers. Nothing is owed here and the diff touches nothing of it.
- **Added (arbiter): the Discussion review's round 2 was accepted at the hard cap.**
  - This is a record-only ruling and asks for no code.
- **Added (arbiter): the split review's round 2 was accepted at the hard cap.**
  - The `fixtries` move it names belongs to adopt-foreground and machine-registry-holds-catchup. This part owes nothing of it.

**Changes**

- **C1: the widened leftover teardown, built on landed and cancelled kinds, traces to G1 and G7.**
  - `internal/dobbyengine/leftover.go:298-329` (`leftoverKind`, landed and cancelled cases) and `tearDownLeftovers`.
  - `internal/beamcli/dobbydeps.go:409-446` (`tearDownLeftovers`, `tearDownLeftover`, `reconcileEnded`, `RegenerateHubWorkspace`).
  - The `LeftoverEnded` kind keeps the earlier ended-run behaviour, which D7 says this teardown widens.
- **C2: the new RunView fields `OwnedElsewhere` and `BoltHeld` trace to G1.**
  - They are filled in `internal/battencli/dobbyreads.go:205` and supply the hold and other-owner exclusions.
- **C3: the hand-over kind traces to G2.**
  - `RunView.HandOverMarked` is read at `dobbyreads.go:223-226`.
  - The `LeftoverHandedOver` branch is at `leftover.go:311-313`.
  - The new function is `pairteardown.RemoveHandedOverPair` (`internal/pairteardown/leftover.go:18-27`).
- **C4: kept pairs trace to G3.**
  - `RunView.PairKept` and `KeptSince`, and `keptSince` (`dobbyreads.go:204,271-282`).
  - `View.Kept` and `KeptPair` (`view.go`).
  - `DobbyLines` kept lines (`overviewdobby.go:206-212`).
  - `noticeKeptPairs`, and `keptNoticeOwed` set at `loop.go:241`.
- **C5: the pre-upgrade kept-cancel marking traces to G4.**
  - `battencli.MarkKeptCancels`, `shedrun.WriteKeptCancel` (`intents.go:69-88`), `daemon.markKeptCancels`, and the `Deps.MarkKeptCancels` wiring.
- **C6: orphans trace to G5 and the Exports line.**
  - `internal/dobbyengine/orphan.go` and `battencli.DobbyOrphans`.
  - `fabricengine.PairLastCommit` and `OrphanFirstSightPath` (`internal/fabricengine/orphan.go`).
  - `lymdirs.OrphanDirName` and `pairteardown.RemoveOrphanPair`.
  - The prime-only wiring at `dobbydeps.go:366-368`.
- **C7: the tender resume gate traces to G6.**
  - `tenderengine.Deps.PushDeferred` and the `pairsToPush` filter (`resume.go:303-305`).
  - `fabricengine.PairPushDeferred` (`pendingflags.go:43-54`), the `tendercli/deps.go:142` wiring, and the help and doc text.
- **C8: the done-run clear now skips a run whose pair still stands. This is forced by C1.**
  - At `donerun.go:117`, without the skip a landed run's records would be cleared before the leftover teardown reads them.
- **C9: the ended-to-leftover renames are forced by C1.**
  - The renames are in `loop.go` (fields and Deps), `ended.go` (deleted) and `ended_test.go` (folded into `leftover_test.go`).
- **C10: the doc updates are forced by C1 through C7.**
  - `internal/dobbyengine/doc.go`, `internal/beamcli/doc.go`, `tendercli/doc.go`, `tenderengine/doc.go`, and the godoc in `dobbyreads.go`.
- **C11: the new bullet in `pattern/PATTERN-pair-remote.md:56-60` is forced by C1, C3 and C6.**
  - The rule doc names the teardown routes, and the dobby is a new pairteardown caller. No PATTERN change is declared, and the bullet does not change the rule's meaning.
- **C12: the tests are forced by the traced changes.**
  - `leftover_test.go`, the `intents_test.go` kept-cancel row, the `resume_test.go` deferred-push row, and the `pause_test.go` `pushDeferred` env seam.

**Findings:** none. Every change traces to a goal point or is forced by one that does. Every goal point and export is built. The Added after Discussion entries govern other parts of the Sketch, or are rulings that ask for no code.

**Tool calls:** none made. The inherited context held the diff, the board entry and the record.
