---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-board-label-maps.md
    summary: Card 3 drops undecided from the template but no card owns the boardcli verbs_test.go fixtures that rely on that template default, so those tests break from card 3 until card 6.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/00-overview.md
    summary: D13 asks for docs/overview.md edits where it lists board or config behavior, but only card 12 edits it (picker removal); the open-map --set and board labels behavior has no overview assignment.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-config-set-open-maps.md
    summary: Cards 2 and 5 do not state D3's bounds in their Intent (only declared paths change, scalar-only Set, a mistyped label name creates a new label); only card 1 does.
  - id: F4
    severity: NIT
    class: scope
    location: _lyx/plan/12-config-bare-and-bogus.md
    summary: With the picker gone, dispatch's in io.Reader parameter and runConfig's os.Stdin argument are dead, and the card does not say to drop them.
---

### [MEDIUM] verbs_test.go fixtures depend on the template's `undecided`

**Location:** `_lyx/plan/03-board-label-maps.md`, `_lyx/plan/06-board-labels-verb.md`
**Issue:** `internal/boardcli/verbs_test.go` (`TestCLIListAndFindLabelFilter` and the `--text` listing test) upserts entries labelled `undecided`.
`seedCwd` writes a `board.yaml` without `labels`, so the label comes from the template default that card 3 removes.
Card 3 greps `undecided` only for fixtures that configure it themselves and edits `internal/boardcli/intake_test.go` but not `verbs_test.go`.
Card 6 edits `verbs_test.go` but its Intent only adds the `labels` test and changes a help example.
After card 3's commit those integration tests refuse the label, and no card is named as fixing them.
**Fix:** Card 3 takes `internal/boardcli/verbs_test.go` and `cli_test.go`'s `seedCwd` into its Edit list and says to give those fixtures their own `labels` (or switch them to a configured label).

### [LOW] `docs/overview.md` assigned only for the picker

**Location:** `_lyx/plan/00-overview.md`
**Issue:** D13 lists `docs/overview.md` where it lists board or config behavior.
Card 12 edits it for the picker, but the `--set` line (open maps) and the board entry description (labels as maps, `lyx board labels`) are assigned to no card.
**Fix:** Card 5 edits the overview's `--set` line, and card 6 its board entry, each in one sentence.

### [LOW] Cards 2 and 5 omit D3's stated bounds

**Location:** `_lyx/plan/05-config-set-open-maps.md`, `_lyx/plan/02-configengine-open-maps.md`
**Issue:** Card 5 is the user-facing route that creates a label, and its Intent never says that a mistyped name creates a new label, that `Set` under an open map writes scalars only, or that undeclared keys keep their refusal.
Card 2 likewise leaves the declared-paths-only bound implicit.
**Fix:** Add a short Bound paragraph to each, pointing at card 1's rule.

### [NIT] Dead `in` parameter after the picker goes

**Location:** `_lyx/plan/12-config-bare-and-bogus.md`
**Issue:** `dispatch`'s `in io.Reader` existed only for `menu`; the card removes the menu branch but leaves the parameter and the `os.Stdin` argument in `runConfig`.
**Fix:** Name dropping the parameter and its callers (tests included) in the card.
