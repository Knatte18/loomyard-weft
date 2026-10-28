```yaml
lens: forced-edits
outcome: reported
```

Lens forced-edits: what each card's change forces outside the members it names.

**MEDIUM — Card 3 (D1), forced edit no card lists: `internal/configsync#TestReconcile_MovesKeysAcrossModules`.**
- Its fixture `burlerOldTemplate` stands for an old-template `burler.yaml` by cutting the template's `twill-discussion` fan to `[:6]` (`internal/configsync/configsync_test.go:2063-2065`).
- That works today only because the fan has seven lenses.
- After card 3 the fan has six, so `[:6]` equals the template and the fixture no longer differs from it.
- The row "a lenses.yaml equal to the template is kept over an old-template burler.yaml and cut to empty maps" (`configsync_test.go:2300-2306`) then tests a template-equal `burler.yaml`. Its premise is gone, and its `kept` and `absent` assertions may fire differently.
- `internal/configsync` is in no card's packages, so card 3's own gate cannot catch this; the batch or webster gate would.
- Fix: card 3 lists `internal/configsync#TestReconcile_MovesKeysAcrossModules` and changes the fixture so it still differs from the new template, for example `[:5]`, or by appending a lens.

**LOW — Card 2 (D6), the overview's Shared Decisions and card 2's Intent overstate what `TestProfilesCheckLoad` guards.**
- They say the row "the template's starting profiles pass" fails on card 2 without card 1.
- That holds only if card 1 is reverted. The rows are already in the tree and card 2 does not edit them, so nothing is wrong.
- But `internal/configreg` is not in card 2's packages (its targets are runprofile, battencli and the template). Card 2's own card gate never runs `TestProfilesCheckLoad`, only the batch or webster gate does.
- The stated "fails on this card" guard is therefore not a check of card 2's own gate.
- Fix: say "fails at the batch gate", or put `internal/configreg#TestProfilesCheckLoad` on card 2 so its package joins the card gate.

Tool calls beyond my context:
1. Read `internal/configcli/configcli_integration_test.go:335-462` and `internal/loomcli/wiring_test.go:900-925`, to check whether those `code_reviewer_fan` uses assert the guard. They don't.
2. Grepped `docs`, `plugins`, `contracts`, `PATTERN.md` and `pattern` for profile and `code_reviewer_fan` wording that cards 1 and 2 might make wrong. Only lines the cards already cover, or lines they leave true, came up.
3. Grepped loom's `template.yaml`, the configreg docs and `runsettings.go` for "closed", "guard" or "local" wording about fan keys. There is none.
4. Grepped tests outside `internal/lenses` for the changed fan and lens names. This found the `twill-discussion` slice in `configsync_test.go`.
5. Read `configsync_test.go` at lines 612-640 and 2050-2090, plus the quarry spans, for the fixture's purpose and the enclosing test glyphs.
6. Grepped `burlerOldTemplate`'s uses, to see which row depends on the fixture differing from the template.
