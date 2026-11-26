```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced the diff's changes to the board entry's goal points.

**Decisions.** The answer key is a decision record alone, with no `Added after Discussion` entries. The board entry under `## Goal` has one expectation, with two alternatives:
- **G1:** a resumed run brings its loom-status strand back.
- **G2:** or `lym reed list` or the run's status says the pane is gone and how to restore it.

**Per change**
- **C1, the restore before the Go-driver spawn:** traces to G1. This is the `restoreStatusPane` field (`internal/loomcli/cli.go:202-205`), the `wire` closure that runs `reedEngine.Up` then `ensureStatusStrand` (`internal/loomcli/wiring.go:1384-1389`), and the call at `internal/loomcli/resume.go:147`. That is the issue's own case: a Go driver spawned after a reboot.
- **C2, the restore before the parked-driver wake:** traces to G1. This is the call at `internal/loomcli/resume.go:174`. "A resumed run" covers both wake paths, and the change description argues this.
- **C3, a failed restore never fails the resume and goes into the envelope as `status_pane`:** traces to G2's fallback. This is `restoreStatusPaneBeforeWake` (`internal/loomcli/resume.go:187-201`) and `paneWarning` in `succeedAfterWake` (`internal/loomcli/resume.go:56-66`). The warning goes into the resume envelope, not into `lym reed list` or the run status that G2 names. G1 is the alternative being built, so this is extra failure-path reporting. It is not out-of-scope work.
- **C4, docs, help and comments:** forced by C1, C2 and C3. This covers `docs/overview.md:1213-1215`, the Resume section of `internal/loomcli/doc.go:110-112`, the `resume.go` file header at lines 6-7, and the `resumeCmd` Long text. The earlier text said resume "never brings reed up" or "adds no strand", which the change made stale. Commit 431dfb100 only rewraps a line from the same edit.
- **C5, tests:** forced by C1, C2 and C3.
  - `TestResumeVerb` gains a counting fake for the restore seam, the `pane` event in the ordered rows, and a failed-restore row (`internal/loomcli/resume_test.go`).
  - `TestSmokeStatusStrandAcrossDriverSeeds` gains a `select-window` setup. The new resume replaces the status strand the earlier subtest added, and the `Loosened-Check:` trailer on c4f96e75a names that cause.
  - `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver` (`internal/loomcli/smoke_resumearming_test.go:25,52`) only has its wording narrowed to "a refused resume". The old wording became false under C1.

No hunk falls outside these five changes.

**Per decision**
- **G1:** built by C1 and C2.
- **G2:** an alternative to G1, so it needs no change once G1 is built. C3 adds a warning for the failure path, but in the resume envelope rather than in `reed list` or the run status.

**Findings:** none.
- Every change traces to G1 or G2, or is an edit a traced change forces.
- No goal point is left without a change that builds it.
- No change edits the task's own board entry.
- The other lenses own whether C3 covers every failure. One example is the replace branch of `ensureStatusStrand`, which logs and returns nil at `internal/loomcli/sharedbootstrap.go:366-369`, so no `status_pane` warning is set.

Tool calls: none; the inherited context held the record, the diff, the commit log and the code.
