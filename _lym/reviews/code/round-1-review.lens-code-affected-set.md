```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, which checks what the diff's changes force outside the lines they touch.

**List: per change, the set found, covered or not**
- **`partsparser.Validate` and `Check` signature (new `worktreeRoot` and `exists` parameters).** Covered.
  - The callers are `loomshed.NewPartsGate` (`partcheck.go:30`), `loomcli/validate.go:426`, `check_test.go:73` and `check_test.go:153`; `partcheck_test.go` was updated.
  - `NewPartsGate` is called only from `shedrecipe/entries_gate.go:212`, and its `requireAbsRoot` check for `WorktreeRoot` has a rejection row in `entries_gate_test.go`.
  - The docs for it are updated: the `entries_gate.go` doc, `loomcli/doc.go` and the `validate-parts` Long text.
- **New `export-existing` finding, plus `handle-malformed` and `fabric-vocabulary-name` emitted from the parts gate.** Covered.
  - `loom-parts-spec.md`'s check table lists all three, and the `Validate` description now mentions the worktree.
  - `partsparser/doc.go` is updated.
  - The Parts-Write stencil's Step 6 mentions both behaviours.
  - Nothing else in `contracts`, `internal` or `docs` lists the parts checks.
- **`ExportMet` narrowed to Create targets and Rename destinations.** Covered.
  - Its callers are `join.go:118` and `partcheck.go:108`.
  - Updated to match: the `join.go` refusal message, the `part-export-missing` detail, `loom-plan-checks-spec.md`, the "Joining the parts' plans" text in the parts spec, the part-writer stencil, and the `Part` doc comment in `parse.go`.
  - The tests `join_test.go` and `partcheck_test.go` were adjusted.
  - The grep for "re-sign or delete" and "creates, re-signs" over `contracts`, `internal`, `docs`, `PATTERN.md` and `pattern` finds no remaining stale statement of the old rule, except the item below.
- **Parts-Write stencil: new Step 4, steps renumbered to 5 and 6.** Covered.
  - The renumbered steps are consistent inside the stencil.
  - Nothing else in the diff or the stencil refers to "Step 4" or "Step 5" by number; the stencil's own "as Step 4 settled" matches.
- **Declared PATTERN changes.** The directive declares none, and the diff touches neither `PATTERN.md` nor `pattern/`.
- **Commit messages.** There is no `Pattern-Departure:` trailer and none is needed.
- **Shared name checks.** `checkHandleMalformed` and `CheckExportName` now share `handleMalformedCheck`, and `fabricVocabularyBreak` is shared. The file-unit rule is not shared; see finding 1.

**Findings**

1. LOW, `consistency`: `contracts/stencils/loom/loom-template-parts.md:38` is left stale.
   - It still says "A part depends on an earlier part when its cards use what that part creates, re-signs or deletes."
   - That is the one place the old rule survives, and it sits beside the new Step 4 rule that an existing member is never an export.
   - The dependency rule is about use, not exports, so it is not wrong, but it reads against the new wording.
   - Evidence: `loom-template-parts.md:38`, against `:51`.

2. LOW, `consistency`: the file-unit rule in `CheckExportName` is not gated the way `validate-plan --part` gates it.
   - `planparser/exportname.go:509` flags a `.go` unit unconditionally.
   - `checkHandleMalformed` at `validate.go:715` gates it on `fileUnitRuleApplies(claims[r])`.
   - A handle that `validate-plan --part` would accept, a Rename-sourced one, is still refused at the parts gate.
   - Only the Create-declaration case that exports use is affected, so this is probably harmless, but the two paths can disagree.
   - Evidence: `exportname.go:509`, `validate.go:715`, `handle.go:188`.

3. LOW, `consistency`: `loom-plan-checks-spec.md` is not updated for the new origin of two existing checks.
   - Its entries for `handle-malformed` and `fabric-vocabulary-name` (not in the diff, so not re-read) do not say that the parts gate also emits them on exports.
   - The parts spec table does say so, and the diff only edited the `part-export-missing` entry there. This is therefore a weak finding.
   - Evidence: `loom-plan-checks-spec.md` hunk at `@@ -181` is the only change in that file.

**Tool calls (1)**
- One grep over `contracts`, `internal`, `docs`, `PATTERN.md` and `pattern`. It looked for stale "re-sign or delete" and "existing member" wording, and for other callers of `Validate`, `Check` and `NewPartsGate`; my context held only the diff and the files I had read.
