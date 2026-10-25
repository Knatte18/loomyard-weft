# Card 29 — origin-owed-marker

**Edit:**
- `internal/battencli#battenCLI.reconcileResume`
- `internal/battencli#battenCLI.driveRun`
- `internal/battencli#TestRunCmd_ResumeDispositions`

**Create:**
- `internal/shedrun#OriginOwedMarker`
- `internal/battencli#originOwed`
- `internal/battencli#OriginOwed`
- `internal/battencli#TestOriginOwedMarker`

**Intent:**
Record a resume that judged its owner without origin, so the next passes stay full until one reaches origin (D4).
`shedrun.OriginOwedMarker` names the per-run marker in the run's scratch directory (`.lyx/shed/<run>/origin-owed.json`, through `lyxdirs`); `originOwed` is its tolerant JSON record, written through `internal/state`, holding when it was written and the pid that wrote it.
`reconcileResume`, when the reconcile reports origin unreachable and `refuseResumeUnreachable` lets the resume go on, writes the marker before the resume; a failed write refuses the resume with its cause and the re-run as the way forward, since a resume that leaves no marker would let dobby skip the reconcile it owes.
`driveRun` removes the marker after a pass whose reconcile reached origin and that exits 0; a pass that did not reach origin, or exits non-zero, leaves it standing.
The pass records the episode itself, so a hand pass, the detached first pass of `lyx beam land` or `lyx adopt` and a pass dobby spawned all leave the same marker.
`OriginOwed` reports whether the marker stands, for the continue-pass check and for dobby.
`TestOriginOwedMarker` (tier0, parallel) over a hand-populated receiver with a fake reconcile: an unreachable resume writes the marker and resumes; an unreachable resume refused for another owner on the local Bolt writes none; a resume whose marker path cannot be written (a directory stands at it) is refused naming the cause and the re-run.
`TestRunCmd_ResumeDispositions`, which drives `driveRun`, gains the rows for the removal: a pass whose reconcile reached origin and that exits 0 leaves no marker, and an unreachable pass that exits 0 keeps it.

**ImpactSummary:** One marker written on the unreachable resume path and removed on a clean reached pass; the reconcile's verdicts and refusals are unchanged.
