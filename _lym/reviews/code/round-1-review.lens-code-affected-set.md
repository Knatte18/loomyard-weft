```yaml
lens: code-affected-set
outcome: reported
```

I reviewed the code-affected-set lens: everything the diff's changes force outside the lines it touches.

**Per change, the set found and whether it is covered:**

- **`stopRecorded` gains an `interrupted` argument.** Callers are `StopStrand` (`stop.go:158`), `stopStepRunsWhere` (`stepstop.go`), `removeSupersededStrands` (`attach.go:240`) and `stop_test.go:186`. All are updated, and no caller outside the package exists. Covered.
- **`carriedInstructions(key)` becomes `newestPredecessor(key)` plus `carriedInstructions(pred, found)`.** The only caller is `run.go` `start`, and no test calls it directly. Covered.
- **`RunState` gains the `interrupted` and `noResume` JSON keys.** Both are `omitempty`. Every write goes through `updateRunState`, under PATTERN-tolerant-records. No other package reads these run.json keys; the same-named fields in creelengine, tenderengine and hardenshed are unrelated. Covered.
- **New `Runner.RetireInterruptedRuns`.** Called only from `loomcli/gotoresume.go`, and described in the shuttleengine `doc.go`. Covered.
- **`relaunchRequest` and `specFor` now take `gotoAdmitted` instead of `prReviewGotoAdmitted`.** The `relaunchRequest` doc comment and the loomcli `doc.go` are updated. The shedverbs `GotoAdmitted` doc and the shedengine `Admitted` doc are generic and stay correct. Covered.
- **Step stops are now recorded as interrupted.** The `StopStepRuns` doc and the shuttleengine `doc.go` are updated, and `stop_test` asserts the flag. Covered.
- **`Start` resumes an interrupted predecessor's session.** The `Start` doc, the shuttleengine `doc.go` and the summary are updated, and `Spec.ResumeSessionID`'s doc (`spec.go:72-74`) stays accurate. Not covered: the orch role stencil, `contracts/orchrole/orch-role.md:296`, still says a re-call after a resumed halt spawns a fresh agent (see the finding). `README.md:18` and `docs/overview.md` hold no wording about a run's start being fresh or carried over, so they are not left wrong.
- **PATTERN entries.** The diff touches shuttle-provider-seam, tolerant-records and test-speed. The pattern background files hold no carry-over or resume-start wording, so none is stale. The directive declares no PATTERN changes, and no `Pattern-Departure:` trailer is needed.
- **Commit messages.** I read all four commits in the range. They carry no trailers, and none is needed: this is a feature, not a bug fix, and no test check was loosened.

**Findings:**

1. **MEDIUM, consistency.** Location: `contracts/orchrole/orch-role.md:296`, the orch role stencil.
   - Summary: it says "a re-call after an escalation's answer or a resumed halt spawns a fresh agent and sends it again". After this diff, a resumed halt whose row's run died, was waiter-orphaned or was stopped as interrupted resumes the recorded session instead.
   - Evidence: `internal/shuttleengine/run.go:642-649` (`resumeInterrupted` in `start`), `internal/shuttleengine/crashresume.go` (`resumableAfterInterruption`), and the stencil line at `orch-role.md:296`.
   - Effect: the orch is told an agent is fresh when it may be the resumed session.
   - Fix: say that a resumed halt after an interruption resumes the row's recorded session, while a re-call after an escalation's answer, or of a run that finished, spawns a fresh agent.

**Tool calls:**

1. Grepped callers of `carriedInstructions` and `stopRecorded` outside the touched files, `.md` docs naming carry-over, fresh sessions or resume, and the PATTERN resume wording. The manifest held no callers outside those files and no doc search.
2. Grepped README, overview, recipe spec, orch-role and glossary for crash, respawn or resume wording, and other packages for readers of the new run.json keys. Needed to find stale docs and readers of the keys.
3. Read `orch-role.md:286-302` and `docs/overview.md:670-700` to check the re-call and died-record wording the grep surfaced.
4. Grepped docs for "operator instructions" and "predecessor" carry-over wording, to rule out a stale carry-over description.
5. Read the `Spec.ResumeSessionID` doc in `spec.go`, to check whether the field's doc claims an empty id always starts fresh.
