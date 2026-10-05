---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/webstercli/cli_tmux_test.go:3
    summary: the file header comment splits one sentence after the subject, mid-clause, instead of at a clause boundary
  - id: F2
    severity: NIT
    class: consistency
    location: internal/reedengine/ensuresession_integration_test.go:65
    summary: the trimmed Best-effort comment still splits a parenthetical across two lines at a comma that is no clause boundary
---

Review range: `status.json` `product.parent` is `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Verified by running: `go build ./...`, `go vet -tags 'integration tmux llm' ./...`, the `cmd/...`, `reedengine`, `gitkit` and `pattern` test packages, `go test -tags tmux ./internal/webstercli/`, and `TestSmokeSelvageSurvivesCtrlCAtIdlePrompt` with `-tags tmux -count=25`; all pass.

Focus verdicts:

1. Added or moved comments in the 14 listed files: two split sentences remain (F1, F2); every other added line there is one sentence per line with breaks at clause boundaries.
   The hard-wrapped lines that remain in `internal/reedcli/smoke_test.go`, `smoke_proctree_test.go`, `internal/shuttleengine/claudeengine/startup_test.go` and similar are legacy comments that the range edited by one token (`smoke` to `tmux`, a retired pattern name to `PATTERN-test-isolation`); `PATTERN-comment-line-breaks` calls these legacy and forbids a rewrap of untouched lines, so they are not findings.
2. `watchdog.go`, `cmd/testtiming/main.go`, `reexecguard.go` and `reedengine/doc.go`: no sentence is split at a non-clause point.
   Every added comment line that ends without terminal punctuation either ends in a colon, ends a usage block, or ends at `, so ...` / `, and <subject> ...`, which is an independent-clause boundary.
3. `EndSessionByName` and `removeSocketFileOnceGone`: the `EndSessionByName` doc comment states the bounded, best-effort removal ("waits a bounded time ... a file still answered after that stays, with no error"), and the code matches it.
   The elapsed-wait-with-live-listener branch is covered by the `socket still answering after the wait stays with no error` row of `TestRemoveSocketFileOnceGone`; not a defect.
4. `TestSmokeSelvageSurvivesCtrlCAtIdlePrompt` waits through `waitForCondition` until the Selvage pane has non-blank content before sending Ctrl-C, and passes 25 of 25 under `-tags tmux -count=25`; not a defect.

### [LOW] F1 — cli_tmux_test.go header breaks mid-clause

**Location:** internal/webstercli/cli_tmux_test.go:3
**Issue:** The first sentence breaks after "the standalone `run` pre-run" and continues "boots a real reed session ..." on the next line.
The break falls between subject and predicate, which is neither a sentence end nor an independent-clause boundary, so it violates `PATTERN-comment-line-breaks`.
**Fix:** Break after the colon, so the clause "the standalone `run` pre-run boots a real reed session before the plan-validation gate, so the test sits in the `tmux` tier" sits on one line.

### [NIT] F2 — ensuresession_integration_test.go parenthetical split

**Location:** internal/reedengine/ensuresession_integration_test.go:65
**Issue:** The range trimmed this comment and left "(the test under test tore it down itself," on one line and "or never booted at all on a failure path)." on the next.
"or never booted at all" is a compound predicate with no subject of its own, so the comma is not a clause boundary.
**Fix:** Join the two lines into one line.
