```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: tracing the code paths that already write or read the state each decision changes.

**Paths traced, per decision**

- D1, restart after Discussion approval: the Motif block's `undiscussed` gate skips Discussion and resumes at Raddle-Write. Survives.
- D4, Raddle-Write's mode record: the first call records the mode, and later calls and Raddle-File read it from the state (`internal/loomshed/raddlewrite.go:60-91`). Survives a restart.
- D6, the filing write's upsert onto the held entry: `checkRunLocks` refuses it today (`internal/boardengine/runlock.go:164-167`), and D6's new admission covers it. Survives.
- D6, a Raddle-File retry: `raddleFilingHeld` and `isRaddleFiledFrom` key on the discussed flag and the project prefix for `f.Sketch` (`internal/boardengine/raddlefile.go:223-241`). The unconverted entry is not held, and the converted one is. Survives.
- D6, a failed write: `FileRaddle` removes every record file it wrote (`internal/boardengine/raddlefile.go:126-130`), the arras entry's own `decision-<slug>.md` among them. D6 names the restore. Survives as stated.
- D6, loom and batten status writes after the filing: `Board.SetRunStatus` writes a run status on any entry that is not done, a project included (`internal/boardengine/board.go:221-230`). D6's skip covers this path. Its key and lifetime break, see finding 1.
- D6, `lym beam create` on the converted project: `clearMotifRun` matches only recipe `motif` (`internal/battencli/cleanup.go:155`), and D6 extends it. It clears only the status and child-recipe records, never the board's raddle store. Breaks, see finding 1.
- D8, the other callers of `Board.SetStatus(slug, nil)`: the batten cancel claim clear (`internal/battencli/cancelcmd.go:319`), the beam task filing (`internal/beamcli/deps.go:238`), the front-task release (`internal/fronttask/hub.go:46`), the pair-teardown claim (`internal/pairteardown/claim.go:181`) and the upsert's `set_status` (`internal/boardengine/store.go:1190`). Breaks, see finding 2.
- D10, the dependency reader: `Dependencies` already holds a dependency on a plain project open until every landing member is done (`internal/boardengine/dependency.go:10-11`). It reads the family case correctly with no change. Breaks D10's premise, see finding 3.
- D7, `copyRecordToBoard` before the filing writes the arras entry's `decision-<slug>.md`, and the D6 filing then writes the project's whole record. Survives.

**Findings**

1. **MEDIUM, design, D6 (status-write skip).**
   - **Summary:** the skip is keyed on "the run's raddle store", but the only raddle store is the board's per-slug directory, and no path that clears the run's records clears it. So the skip never ends.
   - **What breaks:** after `lym beam create` on the converted project, the beam's own status writes under that slug are still skipped. That is the harm the support log gives for rejecting the body-keyed skip.
   - **Evidence:**
     - `internal/boardengine/raddlestore.go:84`: `RaddleStoreDir` is `<board>/raddle/<sketch>`.
     - `internal/boardengine/raddlestore.go:45-48`: `Holds` matches `entry.Sketch == slug`.
     - `internal/battencli/cleanup.go:155-192`: `clearMotifRun` removes only the status and child-recipe records.
     - A grep finds no clearer of `RaddleStoreDir` outside `boardengine`.
   - **Fix:** name a run-scoped record as the key, one that `ClearMotifRun`'s arras extension removes, or name the step that clears the board store's mark.

2. **MEDIUM, design, D8 (release trigger bound).**
   - **Summary:** D8 says the release fires only from `lym board set-status`, but that verb calls `Board.SetStatus`, which several other writers share with a null status. The record does not place the release at the verb, so an implementation in `Board.SetStatus` or `Store.SetStatus` lets those writers release a family.
   - **Concrete case:** `lym batten cancel` on the arras slug after the filing clears the claim through `hubBoard.SetStatus(slug, status)` on the converted project. That would release every draft member with no operator release.
   - **Evidence:**
     - `internal/boardcli/cli.go:705`: the verb calls `Board.SetStatus`.
     - `internal/boardengine/board.go:212-217`: `Board.SetStatus`.
     - `internal/battencli/cancelcmd.go:319`, `internal/beamcli/deps.go:238`, `internal/fronttask/hub.go:46`, `internal/pairteardown/claim.go:181` and `internal/boardengine/store.go:1190`: the other null-status writers.
   - **Fix:** state that the member release lives in the verb's own path or in a dedicated board method only that verb calls. Name cancel's claim clear on a converted project as no release, and add a criterion for it.

3. **MEDIUM, design, D10 (family outcome).**
   - **Summary:** the rationale "a dependent left on it would start before any of the work it waits on has landed" is false for the family outcome. A dependency on a plain project with landing members stays pending until every member is done, and draft members are landing members.
   - **Effect:** moving `depends_on` onto the last parts adds writes to other entries and a new filing-failure path (a run lock on a dependent) for no gain.
   - **Evidence:**
     - `internal/boardengine/dependency.go:10-11`: "A plain project is done when its status is done, or when it has a landing member and every landing member is done."
     - `internal/boardengine/dependency.go:22`: a member is any task or project whose `project` key names the project.
   - **Fix:** limit D10 to the one-task outcome, and keep dependents on the project in the family outcome, citing the dependency rule.

**Tool calls**
- Read `Store.SetStatus` and the recipe check in `clearMotifRun`: the manifest held neither the status-write validation nor what the motif-run clear removes.
- Read `Board.SetRunStatus` and the mode record in `raddlewrite.go`: needed to know whether post-filing status writes reach a project, and whether the mode survives a restart.
- Grepped the `.SetStatus(` callers: needed every writer that shares D8's release trigger.
- Read the null-status callers in `beamcli/deps.go`, `battencli/cancelcmd.go` and `boardengine/store.go`: needed to confirm which of them write a null status.
