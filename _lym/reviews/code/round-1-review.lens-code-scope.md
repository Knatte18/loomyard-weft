```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I trace each change in the diff to a goal point of the board entry and report any change or goal point with no match.

**Decisions** (the decision record alone is the answer key, and it has no `Added after Discussion` entries):
- **G1:** when a row's session ended without finishing (crash, reboot, killed pane), re-calling the row resumes its recorded session, for every role and engine.
- **G2:** a fresh agent stays for a deliberate restart: a goto, a new review round, a re-call after an escalation's answer, or a session whose transcript is missing.

**Changes and the decisions they trace to:**
- **C1 — resume an interrupted predecessor at start.** `crashresume.go` (`resumableAfterInterruption`, `resumeInterrupted`, the preamble), `run.go` (the `start` wiring, the `Start` doc) and the refactor of `newestPredecessor` and `carriedInstructions` in `carryover.go`. Traces to G1. Its gates (same output files, a refused resume check) also build G2's "new review round" and "missing transcript".
- **C2 — tell an interrupted stop from a deliberate one.** The `RunState.Interrupted` field (`rundir.go`), the `interrupted` parameter of `stopRecorded` (`stop.go`), `true` from `stopStepRunsWhere` (`stepstop.go`), `outcome == running` from `removeSupersededStrands` (`attach.go`), and `false` from `StopStrand`. Traces to G1 (crash and killed-pane cleanup stays resumable) and G2 (stops on purpose start fresh).
- **C3 — set aside a run once it is replaced.** `RunState.NoResume`, `setAside` and `markNoResume`, the `takesPlace` set-aside, and the set-aside of both runs when a resume fails to start (`run.go`). Traces to G2: a later visit to the row never resumes a stale session. This is also forced by C1, which would otherwise resume a superseded session again.
- **C4 — a goto sets interrupted runs aside.** `Runner.RetireInterruptedRuns` (`crashresume.go`), the new `gotoresume.go`, the `retireRuns` seam (`cli.go`), and the wiring in `arm.go` and `arbiterconsume.go`. Traces to G2's goto.
- **C5 — docs.** `shuttleengine/doc.go`, `loomcli/doc.go`, and the doc comments on `stepstop.go` and `attach.go`. Forced by C1 to C4.
- **C6 — tests.** `crashresume_test.go`, `gotoresume_test.go`, and the `stop_test.go` updates. Forced by C1 to C4.

**Decisions and the changes that build them:**
- **G1:** built by C1 and C2, but only for engines with `SessionResumer`, which in practice means Claude. Codex is not built; see the finding below.
- **G2:**
  - Goto: C4.
  - New review round: C1's output-file check.
  - Missing transcript: C1's refusal path through `checkResume`.
  - Escalation re-call: no change of its own. It depends on the escalated run having ended `done`, which `resumableAfterInterruption` never resumes.
  - Stops on purpose: C2's `false`.
  - Stale visits: C3.

Every change traces to a decision or is forced by one. No change is unrequested scope.

**Findings:**

1. **MEDIUM, scope.** `internal/shuttleengine/crashresume.go:72-74` (the `SessionResumer` gate in `resumeInterrupted`).
   - G1 asks for the resume on "every role and engine". The diff resumes only on an engine with `SessionResumer`. Codex runs keep starting fresh after a crash, and the diff makes no change to Codex.
   - Evidence: `codexengine.go:103-104`, where `validateSpec` refuses any `ResumeSessionID` ("Codex sessions cannot be resumed by a new run"). `_lym/landing/summary.md` states that Codex starts fresh and calls it "provider work outside this change".
   - The record carries no `Added after Discussion` entry narrowing G1, so the diff leaves part of the goal undone on its own authority.
   - Fix: either give Codex a resume path, or have the parent record the narrowing with `lym loom decision add`. The second is the parent's call, not the fixer's.

Tool calls: none. My context already held the diff, the record, the board entry, the summary and the Codex `validateSpec`.
