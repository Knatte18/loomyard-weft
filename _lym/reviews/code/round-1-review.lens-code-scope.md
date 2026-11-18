```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points.

**Decisions** (a decision record alone; the record holds no `Added after Discussion` entries):
- G1, issue #145: `lym loom decision add` is either refused while a drift judge runs, or its commits are left out of the judge's snapshot check.
- G2, issue #141: a changed value carries its real line number so line ranges match, or Drift-Expect is told to write key-based entries when the diff reports by key.

**Changes and what they trace to:**
- C1, the new `Deps.DecisionRecord` field (`deps.go`) and its wiring in `driftDeps` (`driftwiring.go:249`): traces to G1.
- C2, the record copy in `TakeJudgeSnapshot`, `recordsCheckoutPath`, `appendedDecisionRecord` and the `Changes` exemption (`judgesnapshot.go`): traces to G1.
- C3, the judge-snapshot paragraphs in `doc.go` (lines 252–281 of the new file): forced by C2.
- C4, the three new `TestJudgeSnapshot` rows and the fixture's `DecisionRecord` (`judgesnapshot_test.go`): forced by C2.
- C5, the new sentence in the `seats.go` format text, "Name a keyed line's value by a `keys` entry alone…": traces to G2's second option, telling Drift-Expect to write key-based entries.
- C6, the `checkCase` refusal, the `ParseExpectation` doc lines, the `doc.go` lines 211–212 and the `RangeOverKeyedCase` test row: traces to G2. The refusal reaches the expect agent through the expectation gate, so it enforces the instruction G2 asks for. It is not unrequested scope.

**Decisions and the changes that build them:**
- G1: C1 and C2, with C3 and C4 forced by them. The build is partial (finding 1).
- G2: C5 and C6.

**Findings:**

1. MEDIUM, class `scope`, `internal/driftengine/judgesnapshot.go:270` (the exemption in `Changes`): G1 is only partly built. A `lym loom decision add --amends-plan` still halts the judge.
   - That call also writes and commits the plan amendment record `_lym/loom/amendment.yaml`: `decision.go:779-798` (`beginAmendment`) and `decision.go:1603-1606` (`commitAmendment`).
   - That file is not in the exempt set. `loomRecordsDirs` (`driftwiring.go:290-294`) covers the run dir, the usage dir and the friction dir `_lym/loom/friction` (`config.go:362-363`). The amendment record is `_lym/loom/amendment.yaml` (`config.go:72`, `config.go:308-316`).
   - Darn refuses `--amends-plan` (`decision.go:960-961`), but the tapestry recipe runs Drift-Verify after plan approval (`loomrecipe/doc.go:20`).
   - So in a tapestry run, a parent's `--amends-plan` decision during a judge session still halts the run with `records: …/amendment.yaml was changed`. The issue asks that the verb's commits be left out of the check, not only the decision record's.
   - Fix: exempt the amendment record's appended titles as well, or state the bound in `doc.go` and refuse `--amends-plan` while a judge session is open.

No unrequested-scope finding: every change traces to G1 or G2, or is forced by one that does. The range has no `Pattern-Departure:` trailer.

Tool calls:
- Grepped `LoomFrictionDirRel` and `UsageDirRel` in `config.go` and `paths.go`, and the `--amends-plan` gating in `decision.go`. My context lacked the friction dir's path, which decides whether the amendment record is exempt, and where `--amends-plan` is refused.
