---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttlecli/smoke_interrupt_test.go:302
    summary: five comments this diff added or rewrote still break a sentence mid-phrase or leave a fragment line, against PATTERN-comment-line-breaks
---

Review range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, the base being `d536a1a4e`.

`go build ./...`, `go vet` under the `llm`, `integration` and `tmux` tags, and `go test` over `shuttleengine`, `claudeengine`, `shuttlecli`, `websterengine`, `treadleengine`, `orchcli`, `shedadapters`, `burlerengine` and `contracts` all pass.
`OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking`, `TriageVerdict` and `runTriage` no longer appear in any non-generated file, which is card 12's `Delete` check.

### [BLOCKING] Added or rewritten comment sentences still break mid-phrase or leave a fragment line

**Location:**
- `internal/shuttlecli/smoke_interrupt_test.go:302-304`
- `internal/shuttleengine/wait.go:557-558`
- `internal/shuttleengine/doc.go:51-52`
- `internal/shuttleengine/messagetail.go:1-2`
- `internal/shuttleengine/claudeengine/settings.go:9-10`

**Issue:**
`PATTERN-comment-line-breaks` governs every new or changed Go comment: one sentence per line, with a further break only at an independent-clause boundary.
I derived the added `//` lines with `git diff -U0` over the range and checked each added or rewritten sentence.
Most are clean, and the sentences that only swap or delete words in an untouched wrapped comment are exempt.
Five sites are not:

- `smoke_interrupt_test.go:302-304`, the new doc comment of `newSmokeRunner`, is still column-wrapped.
  It breaks inside "the same way / `shuttlecli.Command()`'s PersistentPreRunE does" and inside "config / it was built from", and its second sentence ("Tests that need the `*Run` handle ...") shares line 304 with the tail of the first.
- `wait.go:557-558`, in the new `pollEventsTick` text, splits one sentence between its subject and its verb: "an EventStop and an EventAsk with output files missing" ends one line and "return a held turn end ..." starts the next.
  The break belongs after the colon.
- `doc.go:51-52` leaves "died, timeout or a mechanism failure." as a one-phrase fragment line, cut from the list it ends in "(every output file present at a turn end, or at the deadline or a pane's death),".
- `messagetail.go:1-2`, a new file header, breaks "sent through / `lyx shuttle send` ends with, and the function that appends it." inside the noun phrase.
- `settings.go:9-10` replaced the final clause of a wrapped sentence, so "loop holds it like any turn end without output." is now a fragment line left over from the old wrap.
  The clause was rewritten, not just a word swapped.

**Fix:**
Rebreak each site at sentence and independent-clause boundaries only, with no change to the wording beyond what the break needs:

- `newSmokeRunner`: put the first sentence on one line (or break it after "does,") and the "Tests that need ..." sentence on its own line.
- `wait.go`: end the line at "whatever its Kind:" and put the "an EventStop and an EventAsk ..." clause on the next line, or make it one line.
- `doc.go`: join "died, timeout or a mechanism failure." to the line above, or restructure so the list sits on one line.
- `messagetail.go`: put the whole header sentence on one line.
- `settings.go`: rewrite the sentence's tail so it sits on one line, and leave the lines above it as they are.

Leave every untouched wrapped line alone, per the pattern.

## Focus settlements

1. Added comment lines: settled in F1.
   Sites such as `singlellm.go:140-144`, `judge.go:2-5`, `judgeverdict.go:2-4`, `template_test.go:61-63` and `driverspec.go:48-52` re-break part of a sentence whose wording is unchanged or only loses words; I treat them as exempt and raise no finding.
2. `lyx shuttle send`: `TestSendCmd_AppendsMessageTail` in `internal/shuttlecli/cli_test.go` runs `sendCmd` over `shuttlefake.Reed` and `shuttlefake.Engine` and asserts the exact text typed into the pane.
   One row pins the tail appended to plain text, the other pins text already ending with `shuttleengine.MessageTail` typed once.
   `TestWithMessageTail` keeps three variants (trailing spaces, a tab, the tail mid-sentence) that the verb test does not cover, so neither duplicates the other.
   Not a defect.
3. `finalize` and `gateLastDone`: `finalize` takes `outcome` alone, every call passes just that, and `gateLastDone` is gone from `run.go`.
   No test compares two values that can only be empty; the remaining `LastAssistantMessage` assertions in `burlerengine/engine_test.go` compare a message the fake supplies.
   `Result.LastAssistantMessage` stays always empty, which card 5 keeps and documents; not a defect.
4. `gofmt -l` over every Go file the range touches lists only `internal/burlerengine/engine.go`.
   The misaligned `Round:` field is on a line this diff does not touch, and `git show <base>:internal/burlerengine/engine.go | gofmt -l` flags the file too.
   It is unformatted because of the base commit, not this diff, so there is no finding.
