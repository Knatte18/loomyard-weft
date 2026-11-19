```yaml
lens: code-scope
outcome: reported
```

Restating my task: the code-scope lens. I trace each change in the diff to the board entry's goal points and file any change or goal point that has no match.

**Decisions**. The record's Scope and Decisions sections are empty, and it holds no `Added after Discussion` entries. The goal points of the board entry are:
- **G1**: `code_review_ask_parent`. Each Code-Review round mails a `[code-review request]` naming the review and fixer report, and waits for `lym loom parent-review answer` before the Bouncer judges the round.
- **G2**: a timeout key for G1, as `discussion_review_parent_timeout_min` has.
- **G3**: `drift_judge_ask_parent`. The judge mails its proposed rulings (every cell, or a summary by case with the flagged cells named) and waits for agreement or changed rulings before it writes the judgement.
- **G4**: a timeout for G3.
- **G5**: both keys settable in `loom.yaml` or `drift.yaml` and in a task's `run_settings`.
- **Out of scope**: routing agent questions to the parent, and parent notification on named rows.

**Per change:**
- **C1**: shedadapters seat generalization: `Tag`, `Subject`, `AskEachRound`, `askSubject`, the new `ParentAskLine` signature, the tag rename to `DiscussionParentAskLineTag`, and `doc.go`. Traces to G1; the rename is forced by the generalization.
- **C2**: loomcli `parentseatwiring.go` (`parentSeats`, `codeParentSeat`, `baseParentSeat`, `askParentSetting(key)`) and `loomSegmentSettings` in `wiring.go`. Traces to G1 and G5.
- **C3**: loomengine `code_review_ask_parent` and `code_review_parent_timeout_min` in config.go and the template, the refusal, and `CapParentReviewWait` taking a perch. Traces to G2 and G5.
- **C4**: driftengine `judge_ask_parent` and `judge_parent_timeout_min` in config.go and the template, the refusal, and the configreg local-keys test. Traces to G4 and G5, and to G3's key.
- **C5**: driftengine `judgeparent.go`, `tableWithParent` and `parentAgreement` in verify.go, the `Deps.JudgeParent` seam, and `doc.go`. Traces to G3.
- **C6**: the `JudgeParent` seat in loomcli `driftwiring.go`, envkit's `driftDeps`, and battencli's `intentionallyNil` row. Forced by C5.
- **C7**: the doc and help text of loomcli `parentreview.go` and `doc.go`. Forced by C1 and C5, since the verbs now serve the new asks.
- **C8**: the two new sections in `contracts/orchrole/orch-role.md`. Forced by G1 and G3: without them the orch has no rule for handling the two new mail tags.
- **C9**: the code keys in `docs/overview.md`. Forced by G1 and G2. The drift keys need no overview line, because the overview documents no drift judge keys.
- **C10**: the tests in driftengine, loomcli, loomengine, shedadapters, configreg and battencli. Forced by C1–C6.
- No change traces to the out-of-scope notes, and no change traces to nothing.

**Per decision:**
- **G1**: C1, C2, C7, C8 and C9, built only in part (F1).
- **G2**: C3 and C9.
- **G3**: C5, C6 and C8, under another key name (F2). Drift-Recheck shares the engine.
- **G4**: C4.
- **G5**: C3 and C4. The code keys are `CategoryNone`, and the drift keys are in configreg's local keys.

**Findings**

F1
- severity: BLOCKING
- class: design
- location: `internal/shedadapters/parentseat.go` `askSubject` / `parentAskDue`; `internal/loomcli/parentseatwiring.go` `codeParentSeat`
- summary: The code-review ask does not do what G1 asks. Each ask goes out at round start, beside the reviewer and before that round's review or fixer report exists. It names only the round before's review and fixer report, and round 1 names neither. The last round's fixer report never reaches the parent. No wait sits between the fixer and the Bouncer, so the parent can never answer a round's review and fixer report before the Bouncer judges that round. `_lym/landing/summary.md` admits this as a "design decision", but no `Added after Discussion` entry licenses it.
- evidence: `parentseat.go` `askSubject` (`round &lt; 2` returns the bare subject; otherwise it names `round-1` paths). `burlerreview.go:295`: "asks the run's parent orch, through roundParentAsk, before the probe and before any spawn". The decision record's Goal names "the review and fixer report … before the Bouncer judges the round". `summary.md` "Design decisions", first bullet.

F2
- severity: MEDIUM
- class: decision
- location: `internal/driftengine/config.go` `configFile.JudgeAskParent` (`yaml:"judge_ask_parent"`), `internal/driftengine/template.yaml`
- summary: The board entry names the key `drift_judge_ask_parent`; the diff ships `judge_ask_parent` in `drift.yaml`. The summary gives a naming rationale, but no record entry settles the departure. The parent should accept it with `lym loom decision add`, or the key should take the entry's name.
- evidence: Diff `005` hunk at `config.go:111` and `template.yaml:718`. The decision record's Goal, `drift_judge_ask_parent` bullet. `summary.md` "Design decisions", second bullet.

Tool calls: none; my inherited context held everything this lens needed.
