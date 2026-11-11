```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changed messages, envelope keys and report fields force outside the lines it touches.

**Per change, the set found**

- **Stuck-reason text.** The text moved from `verify gate failed after N attempt(s) of M` to `verify gate did not pass after N of M re-prompt(s)`. A repo-wide grep finds no other asserter in code, docs, stencils or fixtures. Covered.
- **`VerifyGateReport` fields `Command`, `ExitCode`, and `LogTail` for an opaque failure.**
  - Consumers are `renderVerifyGateFindings`, `writeLogLocation`, and the stuck reason. `writeLogLocation` prints `LogTail`, but only on the timeout, hang, and normal-failure paths. The diff moves an opaque failure's output into `LogTail` and sets `Failures = nil`. That fixer-facing path is affected (see F1).
  - `Command` and `ExitCode` are not rendered in the findings. They are used only by the stuck reason. Not a defect.
- **`notifyLiveRuns` signature.** It has a single caller, `decisionVerb`, and the test drives it through `decisionVerb`. Covered.
- **New envelope key `notice_held_back`.** Described in `internal/loomcli/doc.go` and the websterengine `doc.go`. Not covered in `docs/overview.md:1164` or in the verb's `Long` help text (see F3).
- **Bracket-verb refusal.**
  - All six bracket verbs are wrapped in `cli.go`. The `ownerlessRunWarnings` call sites remain for the non-Master callers, so that path is intact. The webstercli `doc.go` and the Master stencil are updated.
  - The probe-error refusal path is wrong (see F2).
- **Stencils.**
  - `webster-body-implementer.md` adds step 6 and drops no existing rule.
  - `webster-template-master.md` adds two bullets and drops no existing rule.
  - I did not check whether other webster stencils, such as the fork bodies, repeat the amend-vs-new-commit rule.

**Findings**

- **F1 — MEDIUM, design.** `internal/websterengine/verifygate.go:507-397`, with `verifygatereport.go:261-268`.
  - For an opaque failure the gate now sets `report.Failures = nil` and `report.LogTail`.
  - `renderVerifyGateFindings` falls through to `writeFailingIdentities`, which writes a bare `Failing identities:` header with no entries. It then calls `writeLogLocation`, which prints `Log tail:`.
  - The fixer and Merriam therefore read an empty "Failing identities" list followed by the tail. The `Hint` lines also run on a report that names no package.
  - `verifygatereport_test.go` has no row for a tail-only report. Fix: add a branch before `writeFailingIdentities` that, when `len(r.Failures) == 0`, writes "The verify command exited with code N and named no failing test; its output tail follows" and skips the empty list. Add a row to the findings-render test.
- **F2 — MEDIUM, design.** `internal/webstercli/ownership.go:45` and `:59`.
  - `refuseOrphanedMaster` attaches `run_ended: true` to every refusal. That includes the transient probe-error refusal, whose own text says `way forward: re-run the verb`.
  - The Master stencil (`webster-template-master.md:193`) tells Master to end its turn on `run_ended`. A transient lock-read error therefore ends a live run's Master session.
  - Fix: have `orphanedMasterRefusal` return `(message, runEnded bool)`, and set `run_ended` only for the free-lock case. Add a probe-error row to `TestRefuseOrphanedMaster_RefusesAMasterWhoseRunEnded`.
- **F3 — LOW, consistency.**
  - `docs/overview.md:1164` says the verb "reports the strands told under `notified` and any failed notice under `notice_errors`" and omits the new `notice_held_back`. The `Long` help in `internal/loomcli/decision.go:1350-1351` lists only `notified` and `notice_errors`.
  - Fix: add one sentence to each saying a Webster Master session of a run that is not running is named under `notice_held_back` and gets no notice.

Declared PATTERN changes: none declared in the rubric. `PATTERN-refusal-way-forward`: the `run_ended` refusal names a way forward and a test reaches it. The probe-error refusal's way forward has no test (folds into F2's added row).

**Tool calls (6)**
1. `grep` of ownership, envelope and refusal strings across the repo: the manifest had no search for other consumers.
2. `grep` for the old stuck-reason strings: to find other asserters.
3. `renderVerifyGateFindings` and `logTail` definitions: the diff did not show the renderer.
4. `loomshed/verifygate.go` 300-360 and a list of `VerifyGateReport` users: to find other report consumers.
5. `StrandNameEnv` users and docs on decision-add notices: to find other Master-caller paths and docs.
6. `writeLogLocation`, `docs/overview.md` 1155-1170 and the `decision.go` help text: to confirm F1 and F3.
