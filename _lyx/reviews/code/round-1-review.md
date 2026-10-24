---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedengine/goto_test.go:559
    summary: Relaunch entered-row handling (clear on a target, unchanged without one, unchanged when Admit reports the target in effect) has no test
    fix: Add rows to the existing TestRelaunch table in internal/shedengine/goto_test.go that seed Status.EnteredRow and assert it after Relaunch - cleared with a Target not in effect, kept with no Target, kept when Admit reports the target in effect. Reverting the cur.EnteredRow = "" line in Relaunch (internal/shedengine/goto.go) must then fail a row.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/green.go:recordPass
    summary: No test pins that a tree changing during a run records nothing (stillGreen guard), an acceptance criterion
    fix: Add a case to the existing carry integration test in internal/gatecli/test_integration_test.go (or a unit row beside TestGreenRecord) in which the tree changes between the tier passing and recordPass, and assert green.json holds no entry for the tier.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/emptytier_test.go:52
    summary: The D7 filter exemption has a row only for -run; -skip, -list, -bench and the =value, test. prefix and double-dash spellings are unpinned
    fix: Add rows to the existing table in TestEmptyTierFailure in internal/gatecli/emptytier_test.go for args -skip X, -list X, -bench X, -test.run=X and --run X, each expecting no failure line.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/doc.go:200
    summary: The gate.yaml example tier commands still compile only the old tag spellings (integration, tmux, llm), so the tmux-only example at line 210 compiles no tier2 file and would trip the new empty-tier failure
    fix: In internal/gatecli/doc.go lines 200 and 210 (and the llm example if it carries the same shape) spell the example commands as the hub's transitional commands do - tier1 with -tags tier1,integration, tier2 with -tags tier2,tmux, tier3 with -tags tier3,llm - and vet under each tier tag.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/green.go:newestWholeAncestor
    summary: The diff base is the ancestor whole-green entry with the newest Written time, not the newest by history, so re-running a tier on an older checkout widens a later diff base
    fix: In newestWholeAncestor choose among the accepted ancestors the one no other accepted candidate descends from (use the isAncestor seam between candidates), falling back to Written only on a tie; add a row to TestGreenRecord_NewestWholeAncestor with an older commit written later.
  - id: F6
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/green.go:140
    summary: The 20-commit pruning budget counts whole entries with non-empty args, so many -- -run X passes can evict the empty-args entries the diff base chain needs
    fix: In greenRecord.with count only whole entries with empty Args against greenWholeCommitsKept, and keep non-empty-args whole entries only for the current tree.
---

### [MEDIUM] Relaunch entered-row handling is untested

**Location:** internal/shedengine/goto_test.go:559 (behavior in internal/shedengine/goto.go, Relaunch)
**Issue:** D8 and the acceptance criteria require a relaunch with a target to evaluate the target's gate, and one without a target, or with a target already in effect, to re-call the entered row. The diff adds the `cur.EnteredRow = ""` clear, and no test asserts `EnteredRow` after `Relaunch`; reverting the line breaks nothing.

### [LOW] Mid-run tree change recording is untested

**Location:** internal/gatecli/green.go:recordPass
**Issue:** The `stillGreen` guard is an acceptance criterion; the integration tests found cover only the dirty-at-start case.

### [LOW] Filter exemption rows incomplete

**Location:** internal/gatecli/emptytier_test.go:52
**Issue:** `isFilterFlag` and `filterFlags` exempt `-run`, `-skip`, `-list` and `-bench` in several spellings; only `-run` has a row.

### [LOW] gate.yaml doc examples use old tags only

**Location:** internal/gatecli/doc.go:200
**Issue:** The example tier commands name only `integration`, `tmux` and `llm`, which after the retag compile no `tierN` file and would fail the new empty-tier check.

### [LOW] Diff base picked by Written time

**Location:** internal/gatecli/green.go:newestWholeAncestor
**Issue:** D2 asks for the newest ancestor commit; ranking by write time can pick an older ancestor after a re-run on an older checkout. The result is sound, only wider.

### [NIT] Pruning counts argument-carrying whole entries

**Location:** internal/gatecli/green.go:140
**Issue:** Entries that can never serve as a D2 base consume the 20-commit budget.

## Rejected

- Green record directory built from `AnchorPath()` rather than `WorktreePath()` (code-correctness 1): the wait records and `gatecli/doc.go` already name the anchor as the base, and the anchor is the worktree root in practice.
- `rerouteCount` ignoring `skipped` (code-correctness 4): only ungated rows reroute, so a skipped entry never lands in a rerouting row's history.
- `battenshed.countDone` ignoring `skipped` (code-correctness 5): affected-set found the counter serves episode rows, which carry no entry gate.
- `docs/overview.md` omitting the carry (affected-set 2): the overview already names `--gate`, `--guards` and `gate report`; the module table and execution stack are unchanged, so the CLAUDE.md docs rule does not require more.
- `tools/tokencount` fixtures split losing a test (code-tests 4): only a stat-based suspicion, with no lost test shown.

Declared PATTERN changes: all 23 entries land with the declared text (pattern-fit lens, checked against my own read of the `PATTERN.md` diff). No `Pattern-Departure:` trailer in the range. No `_lyx/landing/summary.md` exists, so there is no change description to check.
