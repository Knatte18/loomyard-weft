```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the fixer's change set still agree with the sibling cards, decisions and cross-references it touches?

1. **LOW**: card 4's Intent says nothing about the `Round().Render` assertions that card 3 now pins.
   - Changed text: card 3's Intent sentence "the `Round().Render` assertions of `TestWire_DarnRecipe` and `TestWire_DefaultConfig` expect `lyx gate test --wait --gate code-burler-fix`".
   - Evidence: card 4 deletes `Env.Round` and lists both tests (`_lyx/plan/04-verify-gate-site-key.md:27-28`). Its Intent never says what those tests assert once `Round` is gone, so dropping the assertions card 3 now names is left unstated.
2. **LOW**: card 23's general testdata statement now disagrees with card 28's new exception.
   - Changed text: card 28's added clause "bar `glyphchain/shapes/tagged.go`, whose `integration` constraint becomes `tier1`".
   - Evidence: card 23 still says, with no exception, "Testdata files and fixture source inside string literals are not test files and are left alone" (`_lyx/plan/23-retag-cmd-contracts-tools.md:25`). Card 28 says it applies the rules "as card 23 states the rules".
   - The overview's retag-rules decision now names respelled support files, but it does not name this testdata exception either.

Checked and consistent:
- No card or `lens-review.md` still names the deleted `TestWireSetting_OpenTerminalForcesOpenWindow`.
- Card 31's one-tag scan exempts `TestMain`-only files, so `!tier1` companions pass it.
- Card 31's follow-up list matches the overview's claim that respelled support files survive the follow-up.
- Card 5's `TestDarnTreeScope` key agrees with card 3's `--gate finalize`.
- Card 14's added sentence agrees with the `TestTargetsFor` row it already lists.

Calls made:
- grep of cards 4–6 for `TestWire_` and `wantDarnLandingCall`, and of cards 23–31 for testdata, because my context did not show how later cards treat the readers card 3 now names, or the twins of the testdata rule.
- `sed` of card 31 lines 58–70, because my context lacked the one-tag scan's definition, which decides whether a `!tier1` companion passes.
