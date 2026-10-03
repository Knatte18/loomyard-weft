---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/shedadapters/bouncer_verdict_test.go:1
    summary: new file header is one sentence fixed-column wrapped across three lines
  - id: F2
    severity: LOW
    location: internal/shedadapters/bouncer_circling_test.go:1
    summary: new file header is one sentence fixed-column wrapped mid-phrase across two lines
  - id: F3
    severity: LOW
    location: internal/shedadapters/bouncerfacts_test.go:20
    summary: writeFactsReview doc is one sentence wrapped mid-phrase across two lines
  - id: F4
    severity: LOW
    location: internal/shedrecipe/entries_bouncer_test.go:199
    summary: TestBouncerEntry_EnvJudgeFallback's changed first sentence is column-wrapped and shares its last line with the next sentence
  - id: F5
    severity: LOW
    location: internal/loomengine/review_test.go:65
    summary: TestResolveJudge and TestResolveJudge_MalformedSpec docs each wrap one sentence mid-phrase
  - id: F6
    severity: LOW
    location: internal/loomengine/config_test.go:184
    summary: TestLoadConfig_MalformedJudgeSpec doc wraps one sentence mid-phrase
  - id: F7
    severity: LOW
    location: internal/burlerengine/verdict_test.go:1
    summary: changed file-header sentence stays fixed-column wrapped across four lines
  - id: F8
    severity: LOW
    location: internal/shedadapters/bouncer_clear_test.go:1
    summary: five changed sentences stay column-wrapped, and one changed inline comment holds two sentences on one line
  - id: F9
    severity: LOW
    location: internal/shedadapters/bouncer_commit_test.go:18
    summary: five changed test docs keep their sentences column-wrapped mid-phrase
  - id: F10
    severity: LOW
    location: internal/shedadapters/bouncer_replay_test.go:97
    summary: TestBouncer_Judged_IgnoresFocusFile's changed sentence stays column-wrapped across six lines
  - id: F11
    severity: LOW
    location: internal/shedadapters/bouncer_judge_test.go:1
    summary: changed file-header first sentence stays column-wrapped across five lines
  - id: F12
    severity: LOW
    location: internal/shedadapters/burler_test.go:50
    summary: writeJudgedRound's two changed sentences stay column-wrapped
  - id: F13
    severity: LOW
    location: internal/loomrecipe/fixture_test.go:274
    summary: changed focus-file comment sentence is column-wrapped, not broken at its semicolon
  - id: F14
    severity: LOW
    location: internal/loomrecipe/resume_test.go:294
    summary: TestBounceRouting_BudgetExhaustionBlocks's changed first sentence stays column-wrapped across five lines
  - id: F15
    severity: LOW
    location: internal/loomrecipe/sequence_test.go:26
    summary: two changed sentences (the segment-shape sentence and the scenario-check comment) stay column-wrapped
  - id: F16
    severity: LOW
    location: internal/shedadapters/attachprobe_test.go:164
    summary: changed sentence about the verdict on disk stays column-wrapped across three lines
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`.
`_lyx/shed/review-finding-class/status.json` gives `product.parent: main`, and there is no `_lyx/loom/rework/` directory, so there is no live rework round.

Every finding is against `CONSTRAINTS.md`'s Comment Line-Break Convention: one sentence per line, with further breaks only at an independent-clause boundary, for every new or changed Go comment.
The convention says "Go comments, doc and inline alike" and does not exempt test files.
A changed sentence is governed in full: changing one word in a column-wrapped sentence makes the whole sentence a changed comment.
Untouched sentences that no changed line crosses stay out of scope and are not rewrapped.
Each finding is LOW: the comments are correct in content, and no behavior depends on their layout.

Comments that already comply were checked and are not findings:
`internal/loomrecipe/fixture_test.go`'s `blockBouncerJudge` (break at `, so` before an independent clause), `internal/shedadapters/burler_test.go`'s surviving-round inline comment (break at a colon before an independent clause), `internal/shedrecipe/entries_bouncer_test.go`'s `judgeSeamShuttle`, `internal/loomcli/wiring_test.go`, `internal/loomcli/circling_integration_test.go` and `internal/shedadapters/focus_test.go`.

Regression check on the round-1 fix commits:
`gofmt -l` over every added or modified Go file in the range prints nothing,
and `go build ./...`, `go vet` over `./internal/... ./cmd/... ./contracts/...`, `go test ./...`, the plan's `go test -tags integration` and its `go vet -tags smoke` all pass.
`circlingStatusReader` (`internal/loomcli/circling.go:149`) decides absence from the status file and creates the lock directory before a strict read, the same pattern `internal/loomcli/approve.go:148` already uses, and `TestCirclingStatusReader` covers both cases.
The `SegmentBounces` entry in envkit's `nilLegal`, the non-test comment reflow, and the `CONSTRAINTS.md` tight list introduce no regression.
A spot-check of the non-test changed comments found none still column-wrapped.

### [LOW] bouncer_verdict_test.go file header is column-wrapped

**Location:** internal/shedadapters/bouncer_verdict_test.go:1
**Issue:** The new header is one sentence broken mid-phrase after "the legacy" and "the three".
**Fix:** Put the sentence on one line.

### [LOW] bouncer_circling_test.go file header is column-wrapped

**Location:** internal/shedadapters/bouncer_circling_test.go:1
**Issue:** The new header is one sentence broken mid-phrase after "the Awaiting Reason it".
**Fix:** Put the sentence on one line.

### [LOW] writeFactsReview doc is column-wrapped

**Location:** internal/shedadapters/bouncerfacts_test.go:20
**Issue:** One new sentence is broken after "choosing the verdict the".
**Fix:** Put the sentence on one line.

### [LOW] TestBouncerEntry_EnvJudgeFallback doc is column-wrapped

**Location:** internal/shedrecipe/entries_bouncer_test.go:199
**Issue:** The changed first sentence runs over four lines with breaks mid-phrase ("takes env.JudgeModel/JudgeEffort/" then "JudgeVersion"), and its last line starts the next, untouched sentence.
**Fix:** Break the first sentence at its colon and semicolons only, and start the following sentence on its own line, leaving that sentence's legacy wrap otherwise unchanged.

### [LOW] TestResolveJudge docs are column-wrapped

**Location:** internal/loomengine/review_test.go:65
**Issue:** `TestResolveJudge` (line 65) and `TestResolveJudge_MalformedSpec` (line 87) each break one new sentence mid-phrase.
**Fix:** Put each sentence on one line.

### [LOW] TestLoadConfig_MalformedJudgeSpec doc is column-wrapped

**Location:** internal/loomengine/config_test.go:184
**Issue:** One new sentence is broken after "fails loud at load".
**Fix:** Put the sentence on one line.

### [LOW] verdict_test.go header stays column-wrapped

**Location:** internal/burlerengine/verdict_test.go:1
**Issue:** The diff changed lines 3 and 4 of the single header sentence, but the sentence stays wrapped mid-phrase across four lines.
**Fix:** Put the sentence on one line.

### [LOW] bouncer_clear_test.go changed sentences stay column-wrapped

**Location:** internal/shedadapters/bouncer_clear_test.go:1
**Issue:** These changed sentences stay column-wrapped: the file header (line 1, even splitting "already-" from "judged"), `layoutApprovedGeneration`'s first sentence (line 25), `TestBouncer_Clear_AfterCommitFailureSubsequentCallClears` (line 295), `TestBouncer_Clear_EndToEndSequence` (line 352), and `TestBouncer_Clear_LogsBeforeDiscardingTheApprovedGeneration`'s "The clear is not cheap" sentence (line 447), which starts mid-line and whose last line also starts the next sentence.
The changed inline comment at line 375 holds two sentences on one line.
**Fix:** Put each changed sentence on its own line, breaking only at independent-clause boundaries, and split line 375 into two lines.
Neighbouring untouched sentences keep their legacy wrap and only move to start their own line.

### [LOW] bouncer_commit_test.go changed docs stay column-wrapped

**Location:** internal/shedadapters/bouncer_commit_test.go:18
**Issue:** The docs of `TestBouncer_Commit_ApprovedCallsExactlyOnce` (line 18), `TestBouncer_Commit_BlockingNeverCalls` (line 44), `TestBouncer_Approve_CalledBeforeCommit` (line 175), `TestBouncer_Approve_NilStillCommits` (line 210) and `TestBouncer_Approve_BlockingNeverCalls` (line 277) each had a verdict word changed but keep their sentence broken mid-phrase.
**Fix:** Put each changed sentence on one line, or break at the colon before an independent clause; on line 175 start the following sentence on its own line.

### [LOW] TestBouncer_Judged_IgnoresFocusFile doc stays column-wrapped

**Location:** internal/shedadapters/bouncer_replay_test.go:97
**Issue:** The single sentence, with two changed lines, stays wrapped mid-phrase across six lines.
**Fix:** Break it only at its colon and its `, so` clause boundary.

### [LOW] bouncer_judge_test.go header stays column-wrapped

**Location:** internal/shedadapters/bouncer_judge_test.go:1
**Issue:** The changed first line belongs to a sentence wrapped mid-phrase across five lines.
**Fix:** Put the sentence on one line, or break it at the colon; the following untouched sentence already starts its own line.

### [LOW] writeJudgedRound doc stays column-wrapped

**Location:** internal/shedadapters/burler_test.go:50
**Issue:** Both sentences have a changed line and both stay wrapped mid-phrase.
**Fix:** Put each sentence on one line, breaking only at independent-clause boundaries.

### [LOW] fixture_test.go focus-file comment stays column-wrapped

**Location:** internal/loomrecipe/fixture_test.go:274
**Issue:** The changed sentence runs over three lines, breaking after "focusPath(runDir, round+1)" and "only matters when" rather than at its semicolon.
**Fix:** Break at the semicolon and the `, so` clause boundary.

### [LOW] TestBounceRouting_BudgetExhaustionBlocks first sentence stays column-wrapped

**Location:** internal/loomrecipe/resume_test.go:294
**Issue:** Line 296 changed, and its sentence spans lines 294–298 with mid-phrase breaks.
**Fix:** Break the sentence only at its `, and asserts` boundary; the later paragraphs are untouched and stay as they are.

### [LOW] sequence_test.go changed sentences stay column-wrapped

**Location:** internal/loomrecipe/sequence_test.go:26
**Issue:** The "Each segment contributes" sentence (lines 26–33, changed at 32) starts mid-line after "derived." and ends mid-line before "With the standalone validate rows gone".
The scenario-check inline comment at line 164 is one sentence, changed at its last line, wrapped across five lines.
**Fix:** Put the segment-shape sentence on its own line, and have its neighbours start their own lines without being rewrapped otherwise.
Break the scenario-check sentence at its colon and `, and the fake shuttle` boundary.

### [LOW] attachprobe_test.go changed sentence stays column-wrapped

**Location:** internal/shedadapters/attachprobe_test.go:164
**Issue:** The sentence ending "would go unexercised" has a changed line and stays wrapped across three lines.
**Fix:** Put the sentence on one line, or break it at its `, so` boundary.

## Focus departures

The focus directive named a list of test files; F16 (`internal/shedadapters/attachprobe_test.go`) is outside that list.
It is the same defect the directive targets, a changed sentence straddling an APPROVED-to-CONVERGED line, so the rubric's comment-convention dimension requires it.
