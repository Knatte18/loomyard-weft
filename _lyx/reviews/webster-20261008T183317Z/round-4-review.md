---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/shuttleengine/run_test.go:1176
    summary: two table rows added to the confirm-submission tests are not gofmt-aligned, so gofmt -l lists the file although it was clean at the merge base
---

### [NIT] Two added test-table rows are not gofmt-aligned

**Location:** `internal/shuttleengine/run_test.go:1176-1177` and `run_test.go:1263-1264`.
**Issue:** The `boxes:` and `wantErr:` keys of the rows "Enters under a clock that never advances stop at the Enter count" and "a text that has not appeared when the window closes names the window and the one typing" are padded to align with a `wantKeys:` line that sits on its own line below, so `gofmt -d` wants a narrower alignment.
`gofmt -l internal cmd` lists `run_test.go`, and the file at the merge base (`8196672a6`) is gofmt-clean, so the branch introduced the drift.
The other files `gofmt -l` lists (`fabricengine/pull.go`, `pattern/doc.go`, `shell/posix.go`, `waiting_test.go`) are not in the diff and are not part of this finding.
**Fix:** Run `gofmt -w internal/shuttleengine/run_test.go`; the two hunks only re-align those four lines.

## Range and method

No `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main` in `_lyx/shed/shuttle-send/status.json`): the ten card commits and the review-fix commits on top of them.
I read the plan's ten cards and read `send.go`, `stop.go` and the `run.go`, `wait.go`, `attach.go`, `hold.go`, `session.go`, `config.go`, `template.yaml` and `engine.go` diffs in `shuttleengine`, the `claudeengine` diff, the burler, shed, webster and loom wiring diffs, and the test files that pin them.

## What was checked

- **Build and tests.** `go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, `go test ./...` and `go test -tags integration ./...` all pass; `go test -race` over `shuttleengine`, `claudeengine`, `websterengine` and `burlerengine` passes; `lyx loom lint-comments --base` reports ok.
- **Per-card mechanical checks.**
  - Card 4 Delete: `sendExtraEnters` has no reference left in code or docs; the one remaining "extra Enter" hit is a test comment describing the window rule.
  - Card 9 Delete: `reedOps` has no reference left.
  - Card 10 Delete: `StrandRemover`, `NewReedStrandRemover`, `RemoveStrandIfLive`, `removeStrandIfLive` (webster's), `BurlerRemover` and `fakeRemover` have no reference left in `.go`, `.md` or `.yaml` files, and `remover.go` and `remover_test.go` are gone.
  - Re-signed symbols (`sendVerified`, `confirmSubmitted`, `abandonStartup`, `burlerengine.New`, `reclaimEntryTimeStrands`, `RemoveRecoveryStrands`) compile with every caller, which `go build` and `go vet` over all tags confirm.
- **`PATTERN-shuttle-stop`.** The only production `RemoveStrand` callers left outside `internal/reedengine` are `shuttleengine` itself, `orchcli` (the orch session's strands, not a shuttle run's) and the loom driver's pane probe in `loomcli`, which is the entry's named exception; burler and webster reach a strand only through `Run.Stop` and `Runner.StopStrand`.
- **`PATTERN-completion-signal`.** `stopOutcome` is the single `runOutcomeStopped` return site and calls `allOutputFilesExist` first; `stop.go`, its return sites and its call site are in the tripwire's audited sets, and the pattern file and `wait.go` header list `stopped` as a negative answer.
- **`PATTERN-shuttle-provider-seam`.** The paste-placeholder shape, `C-u` and the Escape-then-text typing choreography live only in `claudeengine`; `shuttleengine` reaches them through `InputBoxReader.TypeSequence` and `InputBoxClearer`.
- **`PATTERN-transient-stop`.** No new class and no new marking site; `TransientAgentStart`'s comment widens and the mapping in `shedtransient` is unchanged.
- **Docs in the same commit.** `shuttleengine/doc.go`, `claudeengine/doc.go`, `burlerengine/doc.go`, `shedadapters/doc.go`, `websterengine/doc.go`, `PATTERN.md` and `pattern/PATTERN-completion-signal.md` all describe the new behavior.
- **Test economy.** Each new test function covers a behavior no other test covers (the handle-form write failure in `stop_test.go` also pins the restored handle state, which the shared table does not), and the other behaviors land as rows in the existing tables.
  Each new behavior has coverage: the idle wait, the poll-count caps under a frozen clock, the late-settling box, the clear and its refusals, the start-time teardown, the gate's idle boundary and unsent re-prompt (including the count cap), and the stop verb's two forms.
- **Deliberate departures that are consistent with the invariants.** `maxUnsentReprompts` and the `windowPollCap` bounds add attempt-count caps beyond the plan's deadline-only wording; `PATTERN-spawn-observability` ("a retry loop around a real spawn caps attempt COUNT, not only elapsed time") requires them, so they are not findings.

## Prior rounds

Read after this review was saved.
Round 3's two findings are fixed in the code: `awaitSettledBox` fails with `ErrSubmissionNotLanded` for agreeing reads taken past `closeAt`, and `TestStopStrand_UnrecordableStopRemovesNothing` pins the guid form's two refusals.
Rounds 1–3 left no disputed or deferred item, and I found no regression of an earlier fix.
The gofmt drift in `run_test.go` appears in the round-3 fixer report as noted and left alone, which is why it is raised here.
