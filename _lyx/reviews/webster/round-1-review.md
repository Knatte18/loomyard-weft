---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-rubric-webster-review.md:78
    summary: The per-card mechanical-check bullet drops "actually ran" for every card type, though card 17 asked to change only the Rename check
    fix: In the Per-card mechanical check bullet of loom-rubric-webster-review.md, restore the words actually ran and so the first sentence reads Confirm every one of the card's own groups' type-specific mechanical checks actually ran and passed, and keep the Rename sentence as the one carve-out. The Delete sentence can stay.
  - id: F2
    severity: NIT
    class: consistency
    location: internal/pattern/pattern_test.go:138
    summary: declaredBullet uses the action replaces, which the decision-record spec does not allow (adds, changes or removes)
    fix: Change the action in the declaredBullet constant from replaces to changes, keeping the rest of the bullet, so the fixture is a valid declared-change bullet.
---

### [LOW] Rubric drops "actually ran" for all card types

**Location:** contracts/stencils/loom/loom-rubric-webster-review.md:78
**Issue:** Card 17 (D1.9) changes the check for a `Rename` group only, to the outcome the diff shows.
The diff also removes "actually ran" from the sentence that governs every other group type.
That weakens the reviewer's duty to confirm that the Edit, Move, Create and Delete checks were really run, which is a rule the card did not ask to change (the rubric's own Stencil wording clause).
The Delete sentence the diff adds restores the check for `Delete` alone.

### [NIT] Test fixture bullet uses an action the spec does not define

**Location:** internal/pattern/pattern_test.go:138
**Issue:** `declaredBullet` carries the action "replaces".
`loom-discussion-spec.md` defines `adds`, `changes` and `removes`.
`pattern.Directive` renders the line verbatim, so the test passes, but the fixture models a bullet no record can hold.

## Notes on the focus questions

- Cards 15 and 16: `commitLog` in `internal/websterengine/gitwrap.go` reads only through `gitrepo.New(worktree).Log(head, LogQuery{FirstParent: true})`. It stops at `base`, skips commits with two parents, and errors when the walk reaches the root without meeting `base`. Trailer values are trimmed and otherwise verbatim, and `DepartureLines` carries them unchanged into `Digest.Departures`, `digestSummaryLine`, `RenderProgress` and `summary.md`. Not a defect.
- Cards 3, 11, 13 and 14: `Card.Pairs` is the flat union across every Rename group's pairs, so `SymbolRenamePairs` covers each group's targets. `renameToolRefusal` runs in `BeginBatch` and `recoverSpawn`, and both CLI verbs map `ErrRenameToolMissing` to `rename_tool_missing`. Not a defect.
- Cards 4-7: `planparser` parses only the overview's section and `discussionparser` parses only the record's. `loomshed` reaches the record through `discussionparser.ParsePatternChanges`. `CheckAddedDecisionsKept` now also reports `pattern-change-dropped` for appended bullets. Not a defect.
- Card 12: no reference to `kickstart_pack_file` or the inline layout is left outside `RetiredKeys` and its test row. The grep also found the historical `docs/research` files and `tools/forkaudit`, which are out of scope. Not a defect.
- Card 17 and the directive edits: apart from F1, no rule is dropped, weakened or contradicted. The edit-directive exception states its bound once.
- Test economy: no duplicate or per-helper test turned up in the sampled diffs. Every behavior I traced has a covering row.
- Mechanical checks: `lyx loom lint-comments` is clean. `lyx gate test` passes over the changed packages (`websterengine`, `webstercli`, `planparser`, `discussionparser`, `loomshed`, `loomcli`, `loomengine`, `pattern`, `languageconfig`, `shedrecipe`, `shedadapters`, `contracts/stencils`, `cmd/lyx`).
