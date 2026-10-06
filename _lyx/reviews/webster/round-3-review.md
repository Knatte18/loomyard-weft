---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/shuttleengine/run.go:490
    summary: new shuttleengine comments keep a second independent clause after a comma on one line (run.go, doc.go, fakes_test.go, startup_test.go)
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/skillload.go:3
    summary: skillload.go keeps a since-clause and a which-clause with their own verbs on one line
  - id: F3
    severity: LOW
    class: consistency
    location: internal/orchengine/watcher_test.go:921
    summary: orchengine comments keep a which-clause and a comma-spliced second clause on one line (watcher_test.go, doc.go)
  - id: F4
    severity: NIT
    class: consistency
    location: internal/orchcli/session.go:60
    summary: LoadSkills comment keeps a which-clause with its own verb on one line
---

The diff implements all four cards.
`go build ./...` and the non-tagged verify tests pass, and `grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal` is empty, which is card 4's `assert-no-callers` for its Delete group.
The two behavior focus questions settle as not a defect; the comment-break question yields the findings below.

### [LOW] shuttleengine comments keep a second independent clause on one line

**Location:** `internal/shuttleengine/run.go:490` (`settleLoadTurn`: ", which the caller retries once"), `internal/shuttleengine/doc.go` (skill-loading paragraph: ", whose still-missing skills are skipped with a logged warning"), `internal/shuttleengine/fakes_test.go` (`skillReed`: ", so a verified send finds its text"), `internal/shuttleengine/startup_test.go` (`TestStartup_SkillLoadTurns`: ", which is process-global").
**Issue:** `PATTERN-comment-line-breaks` requires a further break at an independent-clause boundary in new or changed comments.
Each of these lines joins a second clause with its own subject and verb after a comma.
**Fix:** Break each line at the comma, so the second clause starts its own line.

### [LOW] skillload.go keeps a since-clause and a which-clause on one line

**Location:** `internal/shuttleengine/claudeengine/skillload.go:3` (file header: ", since the transcript format is a Claude Code internal") and the `classifyWindow` comment (", which end at the transcript end").
**Issue:** Same rule as F1.
**Fix:** Break both lines at the comma.

### [LOW] orchengine comments keep a which-clause and a comma splice on one line

**Location:** `internal/orchengine/watcher_test.go:921` (`TestWatcher_SkillSkipCauses`: ", which is process-global") and `internal/orchengine/doc.go` (reload sequence: "a skills step loads the whole orch skill list in one turn, an optional retry step loads what that turn left missing,").
**Issue:** Same rule as F1; the doc line joins two independent clauses by a bare comma.
**Fix:** Break the test comment at the comma, and give the doc's second clause its own line.

### [NIT] LoadSkills comment keeps a which-clause on one line

**Location:** `internal/orchcli/session.go:60`.
**Issue:** "LoadSkills delegates to Runner.LoadSkills, which types the one-turn load message into the pane through tmux." keeps the which-clause on the sentence's line.
**Fix:** Break the line at the comma.

## Focus settlements

- Retry-step timeout: `tickResuming` in `internal/orchengine/watcher.go` takes `st.ReloadRetry` as the step's skills, so the timeout path skips only that list with cause `timeout` and advances to the pointer with no second typing.
  The table row "silent retry turn skips only the retry skills with no second retry" in `TestWatcher_SkillSkipCauses` reaches it.
  An implementation that skipped the full skill list would fail its `wantSkips: 1` (the full list has more than one skill), and one that retried again would fail the call-count check.
  Not a defect.
- Step constants: no assertion in `internal/orchengine` compares `ReloadStep` with a literal where a named constant exists.
  The one literal, `step: 1` in `TestWatcher_ReloadReadsAnUnreadableStepAsThePointer`, is a deliberate old persisted per-skill index used as input.
  Not a defect.
