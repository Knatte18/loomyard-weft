# Card 21 — batten-resume-reconcile

**Create:**
- `plan:internal/battencli#reconcileRun` -> `func reconcileRun(location *lyxcwd.Location, slug string) (fabricengine.OriginReport, error)`
- `plan:internal/battencli#resumeRefusal` -> `func resumeRefusal(slug string, report fabricengine.OriginReport) error`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#Command`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battenshed/doc.go`
- `internal/battencli#TestRunCmd_ResumeDispositions`

**Uses:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/pairteardown#HubSeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams`

**Intent:** D8's batten resume: `lyx batten run <slug>` over an existing run record runs the run-scope reconcile first.
`battenCLI` gains a `reconcile func(slug string) (fabricengine.OriginReport, error)` seam, which `Command` wires to `reconcileRun` (`ReconcileOrigin` over the run scope of the slug, seams from `pairteardown.OriginSeams` with `HubSeamOptions`); nil reads as no reconcile, which only struct-literal tests build.
The resume has no envelope, so `reconcileRun` logs each entry of the report's mutation record at Info through `internal/logger`, so a branch it moved leaves a trace; the run scope tears no pair down.
`battenPreRun`, when the status file is found, calls it before returning: `resumeRefusal` turns the slug's entry outcome into the refusal: `ended` refuses naming the landing (`done`) or the removal from the board, with `lyx cleanup --apply` from the prime as the way forward, whose hub reconcile tears the ended entry's pair down through D6 (card 20); `stopped` refuses naming the branch, its state and the way forward its state takes (D4), the branch's `BranchOutcome.WayForward`, followed by the hop's own re-entry, `then re-run lyx batten run <slug>`; `active` resumes.
An `Unreachable` report logs a Warn and resumes (Auto-mode assumptions); a reconcile error refuses naming it, with fixing the cause the error names (an auth refusal or a missing remote repeats on a bare re-run) and then re-running `lyx batten run <slug>` as the way forward.
A fresh start (no status file) runs no reconcile: `Add` probes origin for the pair and a parent behind origin costs only Merge-In's merge.
`battenshed/doc.go`'s resume paragraph says so.
Bound: the reconcile runs only on a resume, never on a fresh start, and refuses nothing an `active` entry would do.

`TestRunCmd_ResumeDispositions` gains a fake `reconcile` and rows: an active report resumes and the reconcile ran once; an ended report refuses naming the landing and `lyx cleanup --apply` from the prime; a stopped report refuses naming the branch, its text containing the fake report's `WayForward` verbatim followed by `then re-run lyx batten run <slug>`; an unreachable report resumes; a reconcile error refuses naming the error and, as the way forward, fixing its cause then `lyx batten run <slug>`; the `absent_status_file` row asserts the reconcile never ran.

**ImpactSummary:** One seam on `battenCLI` and one step in the resume half of `battenPreRun`; the fresh-start half and `battenPreStep` are unchanged.

**Commit:** 21: batten resume runs the run-scope origin reconcile
