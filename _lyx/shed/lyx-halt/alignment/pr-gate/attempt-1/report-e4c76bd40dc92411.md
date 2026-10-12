---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: internal/shedverbs/loop.go, internal/alignment/session.go, internal/loomcli/looparm.go, pattern/PATTERN-daemon-wakeups.md
    summary: run-robustness, in flight at Merge-In, changes 24 of the same Go files and the same PATTERN-daemon-wakeups entry as this run, and neither record orders the two landings
    fix: The parent decides the landing order of lyx-halt and run-robustness; the run that lands second merges the first one's main and re-runs its gates over the shared files (internal/alignment/session.go, internal/battencli/arm.go and wire.go, internal/loomcli/landingdeps.go, looparm.go, resume.go, start.go and wiring.go, internal/shedengine/goto.go, run.go and rename.go, internal/shedverbs/loop.go, spec.go and their tests, internal/websterengine/doc.go, contracts/specs/refusal-spec.md, contracts/specs/loom-status-spec.md, pattern/PATTERN-daemon-wakeups.md).
  - id: F2
    severity: MEDIUM
    class: decision
    location: internal/loomcli/landingdeps.go, pattern/PATTERN-leaf-packages.md
    summary: convergence-plan, in flight at Webster, also changes internal/loomcli/landingdeps.go and adds its own section to the PATTERN-leaf-packages entry
    fix: The parent decides which of lyx-halt and convergence-plan lands first; the second merges main and keeps both landingdeps.go changes and both new leaf-package sections.
---

### [BLOCKING] Code and PATTERN overlap with run-robustness

**Location:** internal/shedverbs/loop.go, internal/alignment/session.go, internal/loomcli/looparm.go, pattern/PATTERN-daemon-wakeups.md

**Issue:** `lyx loom peers` lists `run-robustness` (running · Merge-In) changing these files that pull request 590 also changes: `internal/alignment/session.go`, `internal/alignment/session_test.go`, `internal/alignment/seam_enforcement_test.go`, `internal/battencli/arm.go`, `internal/battencli/wire.go`, `internal/loomcli/landingdeps.go`, `internal/loomcli/looparm.go`, `internal/loomcli/resume.go`, `internal/loomcli/start.go`, `internal/loomcli/wiring.go`, `internal/shedengine/goto.go`, `internal/shedengine/run.go`, `internal/shedengine/rename.go`, `internal/shedverbs/loop.go`, `internal/shedverbs/loop_test.go`, `internal/shedverbs/spec.go`, `internal/shedverbs/goto_test.go`, `internal/shedverbs/pause_test.go`, `internal/websterengine/doc.go`, `internal/websterengine/verifygate_test.go`, `cmd/lyx/daemonwakeups_test.go`, `contracts/specs/refusal-spec.md`, `contracts/specs/loom-status-spec.md` and `docs/overview.md`.
Both runs also add a line to the declared-wait list of `PATTERN-daemon-wakeups` (`pattern/PATTERN-daemon-wakeups.md`): this run its pause hold, run-robustness its gate test CPU sampler.
The overlap touches behaviour as well as text: this run reshapes the loop watchdog and the shed loop's child wait in `internal/shedverbs/loop.go`, and the alignment attempt's timeout in `internal/alignment/session.go` (the Merge-Review fix of `alignment-attempt-deadline-ignores-pause-sleep`), while run-robustness's Merge-Review carries `alignment-attempt-json-bypasses-state` against the same alignment package.
Run-robustness's decision record names `lyx-halt`'s `paused_since` in an open risk (a kept key going stale across an older binary's resume), so the two designs already touch.
Neither decision record nor board entry says which lands first or how the second takes the first's change.

### [MEDIUM] Code and PATTERN overlap with convergence-plan

**Location:** internal/loomcli/landingdeps.go, pattern/PATTERN-leaf-packages.md

**Issue:** `convergence-plan` (running · Webster) changes `internal/loomcli/landingdeps.go`, where this run wires the pause geometry into PR-Align and the parent lock's `Covered` seam, and adds a section to `pattern/PATTERN-leaf-packages.md`, where this run adds `internal/pauserecord`'s section.

## Scope against the goal and decision record

Beyond the overlaps, the change reads as in scope:

- D1–D3, D6: `internal/pausecli` (with `lyx resume` as a second root command, recorded as a `PATTERN-cli-cobra` deviation) and `internal/pauseengine`.
- D2, D9: the leaf `internal/pauserecord` (record, holds, sleep detection), its `PATTERN-leaf-packages` section, and the record path in `hubgeom` and `standalonegeom`.
- D4: holds in `gitexec`, `githubclient`, shuttle's run start, `lyx shuttle send`, reed resume, and the start refusals in `loomcli` and `battencli`.
- D5: readers in `battenshed`, `shuttleengine`'s Wait, the shed loop watchdog, `parentlock` and the orch watcher.
- D7: `orchengine/pause.go` and the `PATTERN-orch-pane-single-writer` rewording.
- D8: webster's pause retired, the bare loom and shed pause refused, `pause_after` read by the parent-lock wait.
- D9: awake-time idle, the API-error continue, burler's pending-prompt check.
- D10: the watchdog kill naming itself, and the step tie in shuttle's run record.
- The Merge-Review risk `alignment-attempt-deadline-ignores-pause-sleep` is fixed by `975f6649e`, which falls under D5's readers of idle time.

`PATTERN.md` also shortens four index lines this run does not otherwise touch (`PATTERN-review-round`, `PATTERN-dev-prod-binary-separation`, `PATTERN-quarry-cgo`, `PATTERN-documentation-lifecycle`), which reads as room under the overview's headroom line for the reworded `PATTERN-orch-pane-single-writer` line.
It changes no rule.

Shared bookkeeping, named and not escalated: `cmd/lyx/testdata/test-budget.yaml` (both peers), the module table in `docs/overview.md` (run-robustness), and `PATTERN.md`'s index (both peers).

No parent commits are listed by `lyx loom peers`.

Board: `watchdog-bare-reason` (#566) asks for the watchdog name, limit and silence in the halt note and trace, which D10 builds; it also asks for a final trace line from a `shed step` that ends without a clean exit, which this run does not build, so it is not fully superseded.
`operator-followups` depends on `lyx-halt` and takes the runs overview's `paused_since`, the carried open risk, so it does not conflict.
