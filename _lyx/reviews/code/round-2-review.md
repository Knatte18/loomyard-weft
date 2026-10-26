---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/websterengine/runlevel.go:740
    summary: The new unreadable-record refusal of AddedDecisionTexts is unreachable and untested, because DeclaredPatternChanges reads the same file with the same errors on the line above and returns first
    fix: In Run, call discussionparser.AddedDecisionTexts before discussionparser.DeclaredPatternChanges, so a read error of the record reaches the new refusal first. Add a row to the refusal table of TestRun in internal/websterengine/runlevel_test.go (beside "a decision record with a malformed PATTERN changes section") that sets fx.Deps.Geom.DecisionRecordPath to a directory such as fx.Worktree, which makes os.ReadFile fail with an error that is not ErrNotExist, and asserts the refusal carries the wayForward strings "transient" and "lyx webster run". Keep the message's way forward as it is.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/drivewiring.go:157
    summary: No test fails when the arbiterSeam.JudgesFirst assignment is removed or aimed at another row, because TestArbiterJudgesFirst checks only the predicate and nothing calls goDriverDepsFor
    fix: Add a test in internal/loomcli/drivewiring_test.go that builds the deps through goDriverDepsFor (or extracts the seam assembly into a function it can call) and asserts deps.Arbiter.JudgesFirst is non-nil and returns true for loomshed.NameWebster and false for other loom rows. Fold the existing TestArbiterJudgesFirst rows into it, so the predicate is not pinned twice. Batten's seam is built in internal/battencli/drive.go with JudgesFirst left nil, which needs no test.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/websterengine/template_test.go:840
    summary: Both new master rows anchor on stencil section headings, so a heading edit breaks them although the parent_decisions rendering is unchanged; the none row still quotes stencil prose
    fix: In the row "master renders none for no parent decision", drop the contains anchor "\n\nnone\n\n## A plan-drift". Render the master prompt once with nil decisions and once with []string{"none"} through renderMasterWithDecisions, and fail when the two texts differ. In the row "master renders each parent decision in order under the decision section", drop the two heading strings from inOrder and keep only first + "\n\n" + second, plus the check that the second entry follows the first.
---

### [BLOCKING] The unreadable-record refusal in Run can never fire and no test reaches it

**Location:** internal/websterengine/runlevel.go:740
**Issue:** `Run` now reads the decision record twice, through `discussionparser.DeclaredPatternChanges` (`runlevel.go:736`) and then `discussionparser.AddedDecisionTexts` (`runlevel.go:740`).
Both return nil for an empty path or a missing file and return the raw read error otherwise, so any record that fails the second read has already failed the first, and `Run` returned at `runlevel.go:738`.
The new `webster: read the parent decisions` refusal is therefore dead code.
`PATTERN-refusal-way-forward` asks that a webster refusal names the way forward and that a test reaches it.
No row in `runlevel_test.go` drives this message, and none could while the order stays.

### [MEDIUM] The Webster-only marking is wired but no test pins the wiring

**Location:** internal/loomcli/drivewiring.go:157
**Issue:** The acceptance criterion is that only the Webster row is marked.
`TestArbiterJudgesFirst` pins `arbiterJudgesFirst` alone.
The line `arbiterSeam.JudgesFirst = arbiterJudgesFirst` can be deleted, or point at another predicate, and every test still passes: no test in `internal/loomcli` calls `goDriverDepsFor`, and the `drive_test.go` rows set `JudgesFirst` by hand.
The behavior of the new driver row would then silently stop for Webster, which is the incident this task fixes.

### [LOW] The new master template rows pin stencil headings

**Location:** internal/websterengine/template_test.go:840
**Issue:** The in-order row asserts `## A parent decision reaches the code through your forks` and `## A plan-drift` besides the rendered entries.
The none row asserts `"\n\nnone\n\n## A plan-drift"`.
These are assertions on a stencil's static prose, which the rubric's test economy bullet names; the behavior under test is only that Go fills `parent_decisions` with the entries or with `none`.
The round-1 fix of this row removed one quotation of stencil prose and kept this anchor.

## Checked and found sound

- `PATTERN.md` `PATTERN-shed-stop-chain`: the entry names the arbiter-first exception for a blocked stop at a row the driver's arbiter seam marks, keeps the default order and stays on one line.
  It matches `decideDrive` in `internal/sheddriver/driverules.go`: the new row follows the hand-over and tackler-resume-returned rows, requires the blocked state, and sits above the transient, re-step and tackler rows, so the landing-halt, budget, precondition and pause rows still precede it.
  `driverules_test.go` and `drive_test.go` pin both sides of it.
- `facts` in `internal/sheddriver/drive.go` consults `JudgesFirst` only when the seam and the function are non-nil and the state is blocked, so an unmarked row and a run with no seam reach the tackler as before.
- The Master stencil, `internal/websterengine/doc.go` and `internal/sheddriver/doc.go` carry the stuck-reaches-the-arbiter rule once each and the `parent_decisions` marker is in the required set; the orch role stencil already states that the arbiter judges a driver stop first, so no edit is owed there.
- `AddedDecisionTexts` reuses the shared span helper of `identity.go` and its test covers order, a fenced heading, no entry, a missing record and an empty path.
