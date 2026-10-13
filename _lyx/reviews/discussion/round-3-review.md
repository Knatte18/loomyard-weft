---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:66
    summary: D4's package-unit rule takes the directory from Glyph.UnitPath only, but a Go symbol target in an external test package has the pseudo-unit <dir>_test, which is no directory, so an import-only change to that target's own _test.go file is listed as a deviation
    fix: In D4, after the package-unit sentence (line 68), add a rule for the external-test shape - a symbol target whose unit is an external test package covers the files directly in the package directory planparser's sanctioned external-test unit mapping (goPackageDir) gives - and replace "the unit taken through `Glyph.UnitPath` only" on line 66 with "the unit taken through `Glyph.UnitPath`, and for an external test package through planparser's one external-test unit mapping". State the same in the Bound (line 74). Widen the `PATTERN-glyph-conversion-chokepoint` line under Constraints to name that mapping beside `glyph`'s conversions. Add an acceptance criterion - an import-only change in a `_test.go` file of package `<dir>_test`, under a card whose symbol target is in that external test package, is not listed.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:85
    summary: D5 reads every peer's row from the loom recipe, but the peer set also holds darn runs; a darn peer at a row it shares with loom (Merge-In, Publish, PR-Gate) takes loom's position while one at a darn-only row reads 0, and D5 does not say this is the intended rule
    fix: In D5's sentence on line 85, state which recipe a darn run's row is read against. Either keep the loom recipe and add that a darn run's shared row names take their loom positions and its darn-only rows read 0, as an unknown order. Or read each run against its own recipe, by the run's recipe name, and say a darn run's position compares with a loom run's only at shared row names. Extend the acceptance criterion on line 143 to cover a darn peer.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:89
    summary: D5 escalates a peer whose decision duplicates one of this run's, but its parent-commit test escalates only a change made in a different way, so a landed commit that already does what one of this run's decisions does passes without escalation and without a stated reason
    fix: In D5's landed-parent-commit sentence (line 89), either add "or already makes one of this run's decisions" to the escalation test, matching the peer rule's duplicate case, or add one sentence saying that a commit that makes the same change is not escalated, because Merge-In takes it, and add that case to D5's Bound.
---

### [MEDIUM] D4 has no rule for a target in a Go external test package

**Location:** `_lyx/discussion/decision-record.md:66`

**Issue:** D4 sets one rule per target shape: a file unit covers its one file, and a package unit covers the files directly in the directory `Glyph.UnitPath` gives.
Quarry's `Glyph.UnitPath` returns a Go glyph's unit unchanged.
For a symbol in an external test package, quarry names that unit `<dir>_test`.
As the doc of `goPackageDir` in `internal/planparser/targetdirs.go` says, that is "a pseudo-unit with no directory of its own".
D4 binds the unit to "`Glyph.UnitPath` only", so such a target covers no file.
An import-only edit to the `_test.go` file that holds the targeted test then lands in the deviation list.
That is exactly the false deviation D4 exists to remove.
Test files are where import-only edits are most common.
`PATTERN-glyph-conversion-chokepoint` already sanctions planparser's one external-test unit mapping for this case, but D4's Constraints line and its "only" wording exclude it.
D4 therefore does not yet state one explicit rule for every target shape, which is what this round's first focus question asks.
The file-unit and in-package symbol shapes are stated correctly, and the Bound and the subdirectory criterion match them.

### [LOW] D5 does not say how a darn peer's row is read

**Location:** `_lyx/discussion/decision-record.md:85`

**Issue:** `lyx loom peers` lists every board task that holds a run status.
Darn runs mirror their status to the board through the shared loomcli wiring, and the binary embeds `recipes.DarnRecipe` beside `recipes.LoomRecipe`.
The darn recipe shares several row names with loom: Preflight, Describe, Merge-In, Publish, PR-Gate, Finalize and Friction-Reflect.
Under D5 as written, a darn peer at Publish gets loom's Publish position, while a darn peer at Darn-Write or Darn-Bouncer reads 0.
That second peer is reported as an unknown order, although it is plainly earlier than any loom run at PR-Gate.
The verdict is unchanged either way, so no guard weakens.
Still, the Decision's "furthest along by its recipe row" names one recipe for runs of two, and it leaves the plan writer to guess whether this mixing is intended.

### [LOW] D5's parent-commit test drops the duplicate case the peer test keeps

**Location:** `_lyx/discussion/decision-record.md:89`

**Issue:** For a peer, D5 escalates when "a recorded decision in one contradicts or duplicates one in the other", and Q11 in the support log confirms that a duplicate stays a design conflict.
For a landed parent commit, D5 escalates only when the commit changes the same behaviour or PATTERN entry "in a different way".
A parent commit that already makes one of this run's changes is therefore never escalated.
The record does not say whether that difference is intended.
This round's fourth focus question is otherwise settled: D5 states what is compared (the commit's subject and diff against this run's decision record and diff) and what follows, and line 144 pins it with a criterion.

## Settled focus questions

- D5's row source: `lyx loom peers` reads it from `recipes.LoomRecipe`, which `internal/loomrecipe` parses from the binary, so every hub has the same source.
  Constraints names the CLI change and its doc section (`internal/loomcli/doc.go` # Peers, which exists), and the criterion on line 143 pins it for a hub with no checkout.
  Loom row names are unique, so a 1-based position is well defined.
  F2 is the only gap.
- D2's Bound: it names the Merge-In widening and, separately, the Publish narrowing for a stop of another cause (a rejected push, or a tree dirtied after the verify).
  The criteria on lines 133 and 134 cover both.
  The ownership fields (row, head and write time) exist on battenshed's failure record (`Head`, `WrittenAt` in `internal/battenshed/deps.go`).
- D4's reader-visible loss: line 70 and the Bound's last sentence state that symbol names are shown nowhere once they leave the warnings.
