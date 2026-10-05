---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shedverbs/step.go:127
    summary: the short envelopes' conditional friction and parent_notice keys have no test
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shedverbs/steprecord_test.go:143
    summary: filepath.Join(short.stepsDir,logger.TraceID()+".json") is not gofmt-formatted
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shedrun/paths.go:91
    summary: ParkMarker comment says the step envelope reports scratch_dir, which only the recorded full envelope does now
---

Range: no `_lyx/loom/rework/` exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main`.
The plan's verify commands (`go build ./...`, `go test`, `go test -tags integration`, `go vet -tags smoke`) all pass on HEAD, and `TestEnvFile_DefaultStdinIsDevNull` passes under `-tags integration` with its three rows.

Settled per the focus directive:

- Card 1: `envFileKey` leads both the launch and the resume line through `sh.WithEnv` only, after the fork wrap, so the line stays single-line; `Prepare` passes the one `filepath.Abs` path to both builders.
  `buildSettings` keeps the webster fork guard as its only Bash PreToolUse entry, and the Agent, AskUserQuestion and Stop hooks are untouched.
  No `bashStdin*` identifier survives in code, docs or comments, and both deleted test files go with no dangling reference (`modelpin_test.go`'s `wantSpawnSiteCount` follows the deletion).
- Card 2: every exit path goes through `emitStep`, which renders the full envelope, hands the bytes to `rec.write`, and only then prints; a missing record or `--full` prints the full bytes, so the short line never names a missing file.
  `envelope_path` comes from the recorder's returned path, built from `Spec.StepsDir`.
  `StepEnvelope` and `stepErrFields` are unchanged.
  Nothing outside the updated tests parses `friction_dir` or `scratch_dir` from step stdout, and the old `Long` sentences are gone.
- Card 3: the stencil's invocation block holds exactly `lyx shed step {{.run_id}}`, the `(cd <drive-dir> && lyx` form is gone, and the three named sections each carry their own `envelope_path` sentence.

### [MEDIUM] Short envelopes' conditional keys are untested

**Location:** `internal/shedverbs/step.go:127` (`shortStepEnvelope`) and `:142` (`shortStepErrFields`).
**Issue:** The short success envelope carries `friction` and `parent_notice` only when non-empty, and the short error envelope carries `friction` only when non-empty.
`TestStepCmd_ShortEnvelopeOnStdoutFullEnvelopeInRecord` runs stub rows whose friction and parent notice are empty, so its short key sets never contain these keys.
`TestStepCmd_AfterStep` and `TestStepEnvelope_ParentNoticeConditional` run without a `StepsDir` or on `StepEnvelope` directly, so they print or build the full envelope and never reach the short helpers.
The non-empty branches are behavior the diff adds with no covering test.
**Fix:** Add rows to the existing table in `TestStepCmd_ShortEnvelopeOnStdoutFullEnvelopeInRecord` (not a new test) that run with an `AfterStep` hook returning a non-empty friction on success and on refusal, and a producer result carrying a `ParentNotice` on success, and assert the short key set then includes `friction` and `parent_notice`.

### [LOW] steprecord_test.go is not gofmt-formatted

**Location:** `internal/shedverbs/steprecord_test.go:143`.
**Issue:** `filepath.Join(short.stepsDir,logger.TraceID()+".json")` lacks the space after the comma; `gofmt -l` lists the file and the base commit's version was clean.
**Fix:** Run `gofmt -w` on the file.

### [LOW] ParkMarker comment names the printed step envelope as scratch_dir's source

**Location:** `internal/shedrun/paths.go:91`.
**Issue:** The comment says the park marker sits in the directory the step envelope reports as `scratch_dir`.
The printed step envelope no longer carries `scratch_dir` by default; the full envelope in the record does.
**Fix:** Reword the comment to say the full step envelope (held by the step record) reports it as `scratch_dir`.
