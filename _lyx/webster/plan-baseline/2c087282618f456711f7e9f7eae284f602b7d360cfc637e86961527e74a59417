# Card 13 — loom start and resume refuse a busy driver pane with its own kind

**Create:**
- `internal/loomcli#loomCLI.resumeReportPath`
- `internal/loomcli#driverBusyMessage`

**Edit:**
- `internal/loomcli#loomCLI.resumeParkedDriver`
- `internal/loomcli#loomCLI.runDriverSpawnAndWait`
- `internal/loomcli#loomCLI.resumeFromRunState`
- `internal/loomcli/driverresume_test.go`
- `internal/loomcli/resume_test.go`
- `internal/loomcli/start_driver_test.go`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/shedrun#StartDriverBusyKind`
- `internal/shedrun#ResumeReport`
- `internal/shedrun#ParkMarker`
- `internal/loomcli#driverReportPath`
- `internal/loomcli#driverResumeLine`
- `internal/shuttleengine#ErrSessionBusy`

**Intent:**
Give a resume refused on a drafted or busy driver pane a kind batten can retry on, and make every rerun against one park type the same line (decision D8).

`resumeReportPath(runID)` picks the stop-report path once per park.
It reads `shedrun.ResumeReport`: a file whose first line equals the park marker's current content (the stop report the driver parked on) yields its second line; otherwise it picks a fresh `driverReportPath`, creates its directory, writes `<marker content>\n<report path>\n` and returns it, so a new park, whose marker content differs, never reuses an old path.
`resumeParkedDriver` takes its report path from `resumeReportPath` in place of calling `driverReportPath` itself, so a rerun types the same `driverResumeLine` and shuttle's pending-send record (card 7) matches it; on a landed resume it removes the resume-report file beside the park marker, and `runDriverSpawnAndWait`'s stale-marker removal on the spawn path removes it too.

A send error wrapping `shuttleengine.ErrSessionBusy`, which covers the in-draft and foreign-box failures of card 6, is returned from `resumeParkedDriver` unchanged in kind, and its callers refuse with `shedrun.StartDriverBusyKind`:
`runDriverSpawnAndWait`'s parked-live branch reports `output.ErrFields(out, driverBusyMessage(err, retryStart), map[string]any{"kind": shedrun.StartDriverBusyKind})`, and `resumeFromRunState` refuses with `resumeRefusal{message: driverBusyMessage(err, retryResume), kind: shedrun.StartDriverBusyKind}`.
`driverBusyMessage` names shuttle's reason, which quotes the draft, and the way forward: `clear or send the draft in the driver's pane, then run "<retry>" again, or run "lyx batten run <slug>" from the prime`.
The park marker stays in place, as for every failed resume.
A never-appeared `ErrSubmissionNotLanded` does not wrap `ErrSessionBusy` and keeps today's non-retryable refusal and message.

`contracts/specs/refusal-spec.md` gains this row in the loom section, beside the not-parked rows (`PATTERN-refusal-way-forward`; the way forward names no denied command, `PATTERN-no-denied-recovery`):

```markdown
| start/resume: driver pane busy | `lyx loom start` or `lyx loom resume` finds the parked driver's pane holding a draft, an active operator or a busy session, so the resume line is not typed | transient | clear or send the draft in the driver's pane, then re-run the verb, or `lyx batten run <slug>` from the prime |
```

Tests:
- `driverresume_test.go`'s `TestResumeParkedDriver` rows: a sender failing with an error wrapping `ErrSessionBusy` is not retried, leaves the marker and the resume-report file, and two calls against the same marker send the identical line; a landed resume removes both files; a marker with new content picks a new path.
- `resume_test.go`'s `TestResumeVerb` row and `start_driver_test.go`'s parked-driver test row: a busy send refuses with kind `driver_busy`, a message naming the draft and the verb to rerun, and the marker kept; `TestResumeVerb_FailedDeliveryNamesResume` keeps its never-appeared row non-retryable.
Document the busy refusal and the once-per-park report path in `internal/loomcli/doc.go`.

**ImpactSummary:** loomcli's resume branch, its two calling verbs and one refusal-spec row; the existing refusal texts are unchanged.
