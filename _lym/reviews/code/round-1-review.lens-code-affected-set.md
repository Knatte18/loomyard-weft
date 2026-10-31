```yaml
lens: code-affected-set
outcome: reported
```

**Per-change list (set found → covered or not)**

- `PATTERN-loom-contract` and `PATTERN-shed-stop-chain` index entries: both are in `PATTERN.md` with the declared text. The new entry has no `pattern/` background file and no `[background]` link, which the declaration did not require. Covered.
- Removed `CheckSeed`, `Tapestry-Preflight` and the validator and gate pairs:
  - A grep of non-test `.go`, `.md` and `.yaml` files finds only the retired-row comments and data that belong there (`tapestry-recipe.yaml:63`, `loomrecipe/recipekeys.go:23`, `loomrecipe/doc.go:26`).
  - `pattern/PATTERN-cwd-resolution.md` and `pattern/PATTERN-gate-slot.md` are updated. Covered.
- `DarnDescribeTable`, `DarnRouting`, `NewDarn`, `DarnRetiredRows`, `TapestryRetiredRows` and `describe-darn`: no non-test hit remains. Covered.
- `RecipeDarn` and `RecipeTapestry` name branches in non-test `loomcli`:
  - What remains is the "no seed reads as tapestry" default (`arm.go:331,342`, `arbiterprompt.go:32`, `peers.go:260`, `wiring.go:938`, `sharedbootstrap.go:194`).
  - `peers.go:88-90` looks up retired rows in both recipes by name.
  - Whether `peers.go:88-90` breaches D8 is the code-correctness lens's to judge, not mine.
- Moved keys (`plan_*`, `implementer`) in references:
  - The recipe and Go readers name `tapestry.*` (`tapestry-recipe.yaml:386`, `entries_implementer.go:30`).
  - `llm-model-spec.md:128` and `docs/overview.md:600-643` are updated.
  - `implementer-spec.md` is not, see F1.
- `board-entry.md`:
  - The darn recipe no longer names it.
  - Pawl's recipe and the snapshot writer keep it, as D7 asks.
  - The rubric stencil keeps the board-entry form for pawl. Covered.
- `loom-discussion-spec.md`, `shed-recipe-spec.md` and `docs/overview.md`: updated for the contract, the history check on every record, and Record-Write. Covered.
- `loom-status-spec.md`: not checked, because the tool budget ran out.

**Findings**

F1 — MEDIUM, consistency — `contracts/specs/implementer-spec.md:9-10`
- The diff replaced the sentence that identified the row and left the old line beside the new one.
- Line 9 says "Its `Implementer` engine builds the row with the implementer `tapestry.yaml`'s `implementer` key names, read through `implementer_setting: tapestry.implementer`".
- Line 10 still says the same with `loom.yaml`'s key and `loom.implementer`, which contradicts line 9 and D12.
- Line 9's "Its" now has no antecedent, because "Loom's implementer row is the recipe's `Webster` row, between `Plan-Bouncer` and `Parent-Align`…" was deleted.
- Evidence: `git diff` of that file, a single-line replace, plus the grep output at lines 9 and 10.

**Tool calls**
1. A grep for removed or renamed symbols and rows across `.go`, `.md` and `.yaml` files; my context lacked the stale-reference search.
2. A grep for old-module `module.key` references and `board-entry.md` users; my context lacked the call-site and doc set.
3. A grep for `RecipeDarn` and `RecipeTapestry` branches and Darn-Write doc mentions; my context lacked the remaining name branches.
4. A grep for `implementer_setting` and a diff of `implementer-spec.md` and `shed-recipe-spec.md`; I needed to confirm the spec's moved-key wording.
