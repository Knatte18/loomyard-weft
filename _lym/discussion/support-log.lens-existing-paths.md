```yaml
lens: existing-paths
outcome: reported
```

The task was an existing-paths review of the decision record: trace the code paths that write or read the state each decision changes.

**MEDIUM — D3 with D4: a `done` record with a stopped project ends the suppression while that subject is still owned by another machine.**
D3 ends the record `done` even when a local-commit stop held a project back. D4 suppresses only while the record is unfinished. So for the held-back subject, the orch watcher's owner check (`internal/orchengine/owner.go` `ownerTick`) and the dobby's exit (`internal/dobbyengine/loop.go` `beamOwnedElsewhere`) come back at their next tick. The orch that ran adopt is stopped and the dobby exits, which is the 2026-11-02 finding the board names. That happens before the operator can re-run with `--keep-local` or `--drop-local`. A record that holds a stopped project should stay unfinished, or keep suppressing for that subject, until a re-run settles it.

**MEDIUM — D5 with D3 step `check`: the rule that picks check subjects reads `holds.json`, which D5 stops writing.**
`internal/adoptcli/adopt.go` builds `heldSubjects` from `reportHolds`, around line 352. `checkSubjects` (around line 447) also starts a check for any named subject whose `holds.json` still lists a slug, so a re-run after a failed check checks again. D3 says the check step runs "as today", and D5 says `holds.json` is no longer written, but neither says what replaces that input. Without one, a re-run whose owner write changes nothing starts no check. The record should name the replacement: subjects with a live `adopt`/`adopt-hold` entry, or the record's paused-slug list. `runReports` also reads `heldSubjects`; D5 renames its states but leaves this input unnamed.

**MEDIUM — D11 with D4: on the adopting machine, the archive-and-owner-pause can still fire.**
D11 triggers when "a dobby's probe reads its subject owned by another machine, before it exits". During an adopt, the adopting machine's own dobby reads exactly that between the reconcile and the owner write. D4 stops it exiting, but the record doesn't say D11 is skipped too. If it runs, the dobby archives and owner-pauses the runs adopt is taking over. That writes `OwnerCaller` entries that race the `pause` step's in-place conversion, and the dobby's `liftOwnerPause` (`internal/beamcli/dobbydeps.go:486`) later lifts them on its own. D11 should state that it does not run while D4's record names the subject.

**LOW — D5 with D8: a check process from the old binary still lifts holds.**
A check started by a pre-upgrade adopt, or still running across the deploy, runs the old `greenEnd`. That code lifts `HoldCaller` entries and starts first passes, so the claim that a legacy hold "is likewise lifted only by hand" fails for one check's lifetime. The record should state this bound.

**LOW — D7 with D8: nothing brings up a dobby whose beam step failed.**
Today the green end's `BeamUpWorktree` brought up a dobby whose beam step had failed; D8 removes that. The only way back is the warning naming `lym beam start <slug>`, and the record doesn't say so.

Paths traced, per decision:
- **D2:** `ReadCheckViews` (`internal/adoptcli/checkview.go`) lists `AdoptScratchDir` as subject directories. `adopt.json` and `adopt.lock` sit at the top level and read as having no check record, so it survives. No other reader of `AdoptScratchDir` exists outside adoptcli.
- **D4:**
  - Orch `ownerTick`: survives if the accessor is wired in.
  - Dobby loop probe: survives.
  - `passes.go:191` owner-pause probe: survives, since it is skipped for `Held`.
  - Breaks: the window after a `done` record with a stopped project, and D11 on the adopting machine (both above).
- **D5:**
  - Dobby `passes.go:120`, which skips a `Held` run: survives once `Coverage.Held` also covers `AdoptCaller`.
  - `liftOwnerPause` and the hibernate wake read only `OwnerCaller`: survives.
  - `tenderengine` `Resume`: lifts any caller, so lifting by hand works as intended.
  - `battenshed/innerrun_parentlock.go` and `pairteardown/followup.go` read `OwnerHeld`, which the conversion clears. That is the same as today's `HoldCaller` conversion, so nothing new.
  - `checkSubjects` and `runReports` read `heldSubjects`: breaks (above).
- **D6:** the owner write before the deploy; a failed deploy leaves the runs paused. Survives.
- **D7:** `orchcli.BeamUp` called with `Prepare` by `beam start` under `lym s` reconciles again and owner-pauses only runs owned elsewhere. After the adopt those runs are owned here, so it survives.
- **D8:** `green.go` stops reading `holds.json`. It still removes the fix task and resets the try count, so it survives, apart from the old-binary check (above).
- **D11:**
  - The old owner's `ownedElsewhere` pause path is reused: survives.
  - The cleanup reads archive tags only as weft evidence, and no code deletes `archive/*` tags (the teardown only deletes branches covered by them): survives.
  - Breaks: the trigger on the adopting machine (above).
- **D10:** `runArgvs` stops at the first non-zero exit: survives.

Tool calls:
- A grep for `AdoptScratchDir`, `holds.json` and `heldSubjects` readers, to find what else reads the adopt files; I had seen adoptcli only through its doc.
- A grep for `Held`, `OwnerHeld`, `OwnerPauses` and `liftOwnerPause` readers, to trace who reads the pause callers D5 changes.
- A grep for archive-tag deletion and the dobby's owned-elsewhere code, to check whether D11's tags can be pruned or reach the owned-elsewhere exit.
