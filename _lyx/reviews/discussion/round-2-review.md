---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6, D8
    summary: D6 counts Create, Delete and Rename go.mod changes but not a whole-file Move, so its claim that the verdict is the same before and after the changing card, and D8's claim that its verdict is stable across the run, are false for a moved go.mod
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9 Rationale
    summary: D9's rationale says a fixer disputes a check-blocked fix as a false premise under burler's fix rule, but that rule allows a dispute only when the finding's own premise is false, and D9 never says where a true defect with no admissible fix goes
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 Bound
    summary: D1's bound says the card gate catches every root-module interface assignment site a re-signed method breaks, but the card gate builds tagged test files only in the card's own directories
---

### [MEDIUM] D6 leaves out a `Move` of a `go.mod`, so the before/after sameness claim fails for it

**Location:** `_lyx/discussion/decision-record.md`, D6 Decision and Rationale, D8 Rationale.

**Issue:** D6 derives the nested-module set from `go.mod` files on disk, plus `Create` targets, minus `Delete` targets, and counts a `Rename` pair as both.
The spec also has a `Move` card that relocates a whole file, `contracts/specs/loom-plan-spec.md:126`.
A `Move` card names only the file being moved; its destination is in `Intent` prose, `loom-plan-spec.md:198`.
D6 counts neither side of such a move.
Before the move runs, D6 puts the module at the old directory.
After it runs, the disk puts it at the new one.
So for every card at or after the `Move`, the set changes between the plan gate and a later dispatch.
D6's rationale says counting the changes "makes the verdict the same before and after the card that changes them runs".
D8's rationale says its verdict "is stable across the run".
Neither holds here.
A later card's `Verify:` can pass the plan gate and then fire `verify-nested-module` as a blocking finding at a mid-run dispatch, or the reverse.
For `Rename` the claim does hold for the changing card and every later card.
For an earlier card, the set changes once the later card has run.
That case is harmless, because `ValidateDispatch` (`internal/planglyph/planglyph.go:114`) drops findings on done cards, and a card gate is rendered only once the cards before it are done.
The spec already records the same `Move` blind spot for `path-missing` as an accepted false positive, `loom-plan-spec.md:219`.
D6 neither states this gap nor bounds it.

**Fix:** Either count a `Move` of a `go.mod` in D6, treating its source as a `Delete` and, since the destination is prose, refusing or flagging that card shape, or state the `Move` gap in D6 and D8 as a bound.
That bound says what it lets through: a dispatch-time flip of `verify-nested-module` after a `go.mod` `Move`.
It also says what limits it: the author reorders the cards or uses a `Rename`.
Narrow both rationale sentences to the cases they are true for.

### [MEDIUM] D9's rationale rests on a dispute route burler's fix rule forbids

**Location:** `_lyx/discussion/decision-record.md`, D9 Rationale, last sentence, and D9 Decision, second sentence.

**Issue:** The rationale says a fixer handed a blocked fix "already disputes it on the spec row as evidence of a false premise, under burler's existing fix rule".
`contracts/stencils/burler/burler-step-3-fix.md:14-17` allows a dispute only for "a finding whose premise you show to be false", and never for "disagreement with the rubric".
A proposed fix that a check blocks does not make the defect's premise false.
D9 also keeps a true defect reportable when no admissible fix exists.
That finding cannot be fixed and cannot be disputed, so it can only land in the fixer-report's deferred section and leave the round without converging.
D9 never names that route.
On the focus question: D9 does not tell the reviewer to drop the defect, and its bound, that no defect becomes unreportable, is stated.
The defect is only in the rationale's premise and the unstated route for the no-admissible-fix case.

**Fix:** Replace the dispute sentence with what the fix rule allows.
The `**Fix:**` line is a proposal, so the fixer may apply a different admissible fix.
When none exists, the finding is deferred with the constraining check as its reason, and the segment's stuck route takes it to the operator.
State that route in D9.

### [LOW] D1's bound overstates what the card gate builds

**Location:** `_lyx/discussion/decision-record.md`, D1 Bound, second sentence.

**Issue:** The bound says the card gate's `go build ./... && go test ./...` (D7) catches the interface assignment sites that a re-signed method breaks, "for root-module code".
`go build ./...` compiles no test files.
`go test ./...` compiles untagged test files only.
`cardGateCommand` (`internal/websterengine/cardgate.go:19`) adds `-tags integration` only for the card's own target directories, and never compiles `tmux` or `llm` files or files for another GOOS.
An assignment site in such a file outside the card's directories passes the card gate.
Only the plan-level verify gate catches it, later.
On the focus question: D1 does state what catches the lost interface calls.
The defect is only that the statement is broader than the gate.

**Fix:** Say that the card gate catches assignment sites in untagged code, and that the plan-level `## verify:` gate catches the tagged ones.

## Focus departures

- "D5 skips every directory below the worktree root that holds its own go.mod …": the current D5 does not skip nested modules; it scans them through D3. I judged the current text.
  Verdict: no package-level caller is lost.
  A nested-module file that imports a root package resolves through D3's import-path rule and stays blocking, and D5's bound states that method callers there are informational.
  I checked that loading a nested-module directory from the root does not fail the whole load: `go list -e ./internal/planglyph ./internal/planglyph/testdata/glyphchain/callers` exits 0, with a per-package "main module does not contain package" error, so D5's "leaves unloaded" holds under `go/packages`.
- "D9 says … a fix such a check blocks is not a legitimate finding": the current D9 binds only the proposed fix and keeps the defect reportable.
  I judged the current text, see F2.
- "D6 … while a card's Rename group can move a go.mod": for `Rename`, D7 and D8 agree before and after the renaming card, for that card and every later one.
  The gap I found is the `Move` type, see F1.
