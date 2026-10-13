---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 (file-level comparison), Bound, and the import-only acceptance criterion
    summary: D4 treats a symbol target's glyph unit as a file, but a Go symbol target's unit is its package directory, so the file-level exemption either never matches a symbol target or exempts every file in a targeted package
    fix: In D4, replace "unless it is the file part of a card target, a symbol target or a file-only target alike, taken through glyph's conversions" with an explicit rule for each target shape. For a file-unit target, Glyph.UnitPath gives the file and is compared by equality. For a package-unit symbol target, pick one rule and state it. Either the exemption is directory-level, so every no-symbol changed file directly in the target's package directory is exempt, or the target symbol's file is resolved through the plan index the record-batch already holds and is compared by equality. Rewrite the Bound to name what the chosen rule lets through, which under the directory rule is any no-symbol change to any file in a targeted package, non-Go files in that directory included. Reword the acceptance criterion "An import-only change in a file a card's symbol target names is not listed" to match the chosen rule. Keep the PATTERN-glyph-conversion-chokepoint constraint, which already covers UnitPath.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5, the sentence beginning "The agent reads the row order from the hub's loom recipe file"
    summary: No hub loom recipe file exists; the loom recipe is embedded in the lyx binary (contracts/recipes.LoomRecipe), and only a loomyard checkout carries contracts/recipes/loom-recipe.yaml, so the stencil pointer resolves to nothing in every other hub
    fix: In D5, replace the hub-recipe-file source with one lyx itself serves from the recipe the binary runs. For example, lyx loom peers reports each peer's row position in the loom recipe, or an empty position when its producer is not a loom row. The stencil then reads that field and restates no rows, which keeps PATTERN-producer-pointer-rule. Name the chosen surface in D5 and add it to Constraints as a CLI output change with its doc (internal/loomcli/doc.go, and docs/overview.md if a verb is added). Add an acceptance criterion that the order comes from that surface in a hub with no loomyard checkout. If a new field or verb is chosen, state its Bound, that it is read-only and reveals only peers' row positions.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2, the sentence "The Publish check takes the same match" and the Bound sentence "every other blocked stop keeps its eligibility, a stale record from an earlier verify included"
    summary: D2 narrows today's unconditional Publish refusal to a record that belongs to the halt, so a Publish halt of another cause with a stale record becomes eligible, which contradicts the Bound's keeps its eligibility and has no acceptance criterion
    fix: In D2's Bound, state that the Publish refusal narrows. A blocked Publish stop of another cause, such as a rejected push or a tree dirtied after the verify, with a stale verify-failure record naming a target becomes eligible to the arbiter where it is refused today. Restrict "keeps its eligibility" to Merge-In and the other rows. Add an acceptance criterion that a blocked Publish stop of another cause, with an earlier verify's failure record naming a target still on disk, is eligible.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5, the escalation sentence beginning "It escalates on an overlap only when the designs conflict"
    summary: D5 drops escalation for an overlap with a landed parent commit, but its design-conflict test is phrased over both runs and their recorded decisions, so whether a landed parent commit that changes the same behaviour differently still escalates is left open
    fix: In D5, state whether a design conflict with a landed parent commit escalates. A landed commit has no decision record, so name what the agent compares, for example the commit's diff and subject against this run's decision record. If it does not escalate, add the case to D5's Bound as let through to Merge-In's verify, and add an acceptance criterion for the chosen behaviour.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D4, the sentences on the digest's deviation list and on scope findings leaving record-batch's warnings
    summary: D4 keeps only a deduplicated file list in the digest and removes the scope findings from the warnings, so the symbol names the guard reports reach no reader any more, and the Decision does not say so
    fix: In D4, either keep the symbol on each digest entry (for example a file and its touched symbols) or state in the Bound that the reader surface drops to file granularity and the symbol names the guard reported are no longer shown anywhere.
---

### [BLOCKING] D4's symbol-target file part is a package directory in Go

**Location:** `_lyx/discussion/decision-record.md` D4, its Bound, and the criterion "An import-only change in a file a card's symbol target names is not listed".

**Issue:** D4 exempts a changed no-symbol file when it is "the file part of a card target, a symbol target or a file-only target alike, taken through `glyph`'s conversions", and the focus asks whether a symbol target counts as naming its file.
D4 says it does, but the premise fails in Go.
`Glyph.UnitPath` in quarry's `glyph/unitpath.go` returns the unit unchanged: "the package directory for a package unit, the file for a file unit".
`planparser` stores symbol-shaped targets as glyphs, and a Go symbol glyph such as `internal/loomcli#consumeArbiterAttempt` has a package-directory unit.
The chokepoint conversion therefore gives a directory, not a file, for every symbol target.
Compared by equality, an import-only change in the file that holds a targeted symbol is always listed, which contradicts criterion 126.
Compared by prefix, every no-symbol change to any file in a targeted package is exempt, which the Bound does not state.
The delta cannot settle this either: an import-only file contributes no symbols to `DeltaAnswer.Created`, `Deleted` or `Modified`, so nothing in the delta links that file to the target.
The plan writer has to make a design choice the record presents as already made.

### [BLOCKING] D5's hub loom recipe file does not exist

**Location:** `_lyx/discussion/decision-record.md` D5, "The agent reads the row order from the hub's loom recipe file, which the stencil points at".

**Issue:** The loom recipe is embedded in the binary.
`contracts/recipes/recipes.go` declares `//go:embed loom-recipe.yaml` as `LoomRecipe`, and `internal/loomrecipe` parses `recipes.LoomRecipe`.
The hub's `_board/_lyx/` holds `config`, `specs` and `stencils`, and no recipe.
The only `loom-recipe.yaml` files on disk are the `contracts/recipes/` copies in loomyard checkouts.
The alignment agent runs in every hub, and outside this repository it has no file to read.
Inside it, the worktree's copy can differ from the recipe the running binary executes, for example in a task that edits the recipe.
`lyx loom peers` reports each peer's status as `<state> · <producer>`, as its output on this hub shows, but no surface gives the row order.
The focus question of where the agent reads the order is answered by a source that does not exist, so the furthest-along rule cannot be applied as written.
The rest of D5's Bound is adequate: it names the behavioural text-only interaction left to Merge-In's verify, the same-way PATTERN change, and the no-orch-turn landing at `pr-gate`.

### [MEDIUM] D2's Publish narrowing contradicts its Bound

**Location:** `_lyx/discussion/decision-record.md` D2, "The Publish check takes the same match" and the Bound.

**Issue:** Today `checkEligible` in `internal/loomcli/arbiter.go` sets `PublishHaltNamesTarget` from `PublishFailureTarget`, which reads the record's target whatever its age.
`arbiterEligibility` therefore refuses every blocked Publish stop while any record naming a target is on disk.
D2 makes Publish take the halt-ownership match, so a blocked Publish stop of another cause becomes eligible where it was refused.
`Publish.Call` can halt on a rejected push or a dirty tree after the verify, and the earlier verify's record survives because `clearFailure` runs only after a passing verify and a clean tree.
That narrows a guard, so the rubric's attack-surface rule requires the Bound to name it.
The Bound instead says "every other blocked stop keeps its eligibility, a stale record from an earlier verify included", which is true at Merge-In and false at Publish.
No acceptance criterion pins the Publish case.

On the focus's Merge-In question the widened refusal holds.
`landingshed`'s Merge-In writes the record only from `gate.check` on a failed verify, and that verify failure is the halt.
A later merge-conflict or dirty-parent halt is a new `Stuck` entry after the earlier halt's own entry, so the record's write time is not after the entry before the halt, and `RecordMatchesHalt` rejects it.
Folding cannot merge the two entries: `appendOrFold` folds only budget-exempt Stucks, and a Merge-In gate stop goes through `gateStop` with `halt=false`, so it is not budget-exempt.
The criterion that a merge-conflict or dirty-parent stop with a stale record stays eligible therefore matches the code's match.

### [MEDIUM] D5 leaves design conflicts with landed parent commits open

**Location:** `_lyx/discussion/decision-record.md` D5, the escalation sentence.

**Issue:** D5's first sentence ends the automatic escalation for "an overlap with a peer run or a landed parent commit".
The design-conflict test then reads "both runs change the same behaviour in different ways, a recorded decision in one contradicts or duplicates one in the other".
A landed parent commit is neither a run nor a holder of a decision record.
Whether a landed commit that changes the same behaviour differently still escalates is undecided, and the Bound does not list it as let through.
Today such an overlap escalates, so this is a narrowed guard with no stated bound.

### [LOW] D4 drops the guard's symbol names from every reader surface

**Location:** `_lyx/discussion/decision-record.md` D4.

**Issue:** `ScopeGuard`'s findings name the symbol and its file, and `recordbatch.go` today appends each finding to the warnings.
D4 stores only the deduplicated, sorted file list in the digest's `deviations` (`[]string` in `digest.go`) and removes the findings from the warnings.
After D4 no reader sees which symbol was touched outside the targets.
That may be intended, since the board entry asks for a list a reader sees from the guard, but the Decision neither chooses it explicitly nor states the loss.

## Focus verdicts not raised as findings

- D1: the relaxed pause check does not reach an unmoved run.
  D1 confines it to the verdict-in-effect admit, says "An unmoved run keeps today's equality check", and the third acceptance criterion pins it.
  On the admit itself, `Goto` clears `PauseBefore` and `PauseAfter`, so "present now" can only mean a condition set after the goto.
  A condition re-added with the same value is admitted, but `shedengine`'s run loop honours `PauseBefore` and `PauseAfter` at the producer boundary.
  `Relaunch` runs `CheckGotoTarget` before `Admit`, and for a paused run `gotoAdmitted` admits the current row, so the already-in-effect target passes the target check.
  Not a defect.
- D4's Bound otherwise names what the list lets through and what it reports, unsupported and extraction-error dispositions included, apart from the directory issue in F1.
