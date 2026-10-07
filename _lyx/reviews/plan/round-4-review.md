---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-mark-guards.md
    summary: The candidate set omits roleskills_test.go and parentdirective_test.go, the enforcers of the (test) entries role-skills-typed and parent-directive, and misattributes two stencil scans to the decision.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/09-mark-guards.md
    summary: scanKeeps changes what the keep directive accepts, but cmd/testtiming/main.go, whose package doc states the directly-above rule, is not a target.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/11-tmux-tier-green.md
    summary: The Intent records a point-in-time measurement (the failure did not reproduce when the plan was written, a tmux version, a non-ancestor revision) that bears on no branch choice and goes stale.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/00-overview.md
    summary: The Card Index line for card 11 says any failure the tier shows is fixed, but the card's third branch ends unfinished without a fix.
---

Reviewed `_lyx/plan` (overview and cards 1–15) against `_lyx/discussion/decision-record.md`, the rubric, `PATTERN.md` and the repo.
Every decision D1–D11 and every constraint is carried by some card, and no card carries scope the record does not license.
The remaining findings are narrow.

### [MEDIUM] Card 9's candidate set misses two enforcers of `(test)` entries

**Location:** `_lyx/plan/09-mark-guards.md`, the `**Edit:**` list and the sentence "the two stencil scans the decision names".
**Issue:** The card says the marker goes on "the enforcing tests of `PATTERN.md`'s `(test)` entries", and D5 makes those the starting set.
`PATTERN-role-skills-typed` and `PATTERN-parent-directive` are `(test)` entries, and their enforcers are `contracts/stencils/roleskills_test.go` and `contracts/stencils/parentdirective_test.go`.
Both scan every registered stencil through `Registry()`, which is exactly how the listed `contracts/stencils/citation_enforcement_test.go` and `claims_test.go` scan, yet neither is a candidate.
The card also calls `claims_test.go` and `citation_enforcement_test.go` "the two stencil scans the decision names"; D5 names `contracts/stencils/claims_test.go` only.
Because the card says its list is the candidate set and the acceptance criterion is that every repo-scanning guard carries the marker, an implementer working from the list leaves two guards unmarked.
**Fix:** Add the two files to the `**Edit:**` list.
Reword "the two stencil scans the decision names" so it names `claims_test.go` as the decision's and `citation_enforcement_test.go` and the two role scans as reached through the `(test)` entries and the scankit scans.

### [LOW] `cmd/testtiming/main.go` documents the rule card 9 changes

**Location:** `_lyx/plan/09-mark-guards.md`, `**Edit:**` list (`cmd/testtiming/keep.go`).
**Issue:** `cmd/testtiming/main.go`'s package comment states that the keep directive sits "on the line directly above its `func Test…` line" and that one "not directly above a top-level test" aborts the run.
Card 9 makes `scanKeeps` accept `//lyx:` lines between the keep and the func line, so that comment becomes false.
`cmd/testtiming` has no `doc.go`, so `main.go` is the module doc that the docs-in-the-same-commit rule requires updated.
**Fix:** Add `cmd/testtiming/main.go` to the `**Edit:**` list and name, in the Intent, that its package comment states the stacking rule.

### [LOW] Card 11's Intent records a point-in-time measurement

**Location:** `_lyx/plan/11-tmux-tier-green.md`, the first Intent paragraph.
**Issue:** "That revision is not an ancestor of this tree, the test is now the `CrossClientControl` step of `TestSmokeDotFill`, and the failure did not reproduce when the plan was written" mixes a derivable git fact and a one-time observation with the one clause the implementer needs.
The implementer's branch choice comes from its own first run of the tier, which the card orders first, so "did not reproduce when the plan was written" changes no action and is wrong the moment the tree or the machine changes.
"tmux 3.6" pins a version the same way.
The other sentences earn their place: the recorded message, the absence of a call site, the harness-window facts (checked against `newDotFillHarnessOn`, all callers at 140 by 42, `window-size manual`) and the two helpers (`restartSession` through `mustRunReed`, `addStrandIn`) tell the implementer where not to look and where output surfaces.
**Fix:** Cut "and the failure did not reproduce when the plan was written" and "tmux 3.6", and state the revision claim as "the test is now the `CrossClientControl` step of `TestSmokeDotFill`".

### [NIT] Card Index line for card 11 overstates the card

**Location:** `_lyx/plan/00-overview.md`, Card Index entry 11.
**Issue:** "any failure it shows is fixed, so it passes before Publish gates on it" contradicts the card's third branch, where a cause outside the three test files ends the card unfinished and halts the run.
**Fix:** Reword to "run on the clean tree; a failure whose cause sits in those test files is fixed there, and any other stops the run before card 12".

## Focus questions

1. **Card 11 targets and stop rule — not a defect.**
   The card's `**Edit:**` lists all three files, not only `smoke_dotfill_test.go`: `smoke_dotfill_test.go` holds `newDotFillHarnessOn` and the `CrossClientControl` step, `smoke_lifecycle_test.go` holds `restartSession` and `mustRunReed`, and `smoke_test.go` holds `addStrandIn`.
   Together they cover every site its Intent names as a place the "no space for new pane" text could surface.
   The third branch stops on a cause in reed production code (`internal/reedengine` included) and says the card is not widened, which is the stop condition the directive describes.
2. **No card sets `publish_verify` while the tier fails — guaranteed, with no `Uses` edge needed.**
   The only card that sets it is 12, and its Intent states the precondition ("only after card 11 has finished with the `tmux` tier passing").
   Card 11 carries `**Verify:** go test -tags tmux ./...`, and a fork returns FAILED on a failing per-card verify, so the stop branch fails the card rather than passing it.
   Webster sequences by derived edges and falls back to declared order (`SequenceBatches`: "an already dependency-correct plan sequences to exactly its declared order"); card 11 has no incoming edge, so card 12 cannot be placed before it.
   A `Uses` edge from 12 to 11 would need a symbol or file 11 produces, and 11 may change nothing, so none is available or wanted.
3. **Card 9's `**Edit:**` list is stated as a candidate set — yes.**
   The Intent says the list "is the candidate set to check, not a promise that each file changes", that the criterion alone decides what is marked, that own-package seam and leaf scans stay unchanged, and that `cmd/lyx/crosscompile_test.go` "builds the module through the compiler and scans nothing".
   The set's completeness is F1; its framing needs no change.
4. **Card 11's Intent sentences — one mostly-stale paragraph.**
   See F3; every other sentence bears on where the implementer looks or which branch it takes.
