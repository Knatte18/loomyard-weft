```yaml
lens: fix-claims
outcome: reported
```

fix-claims: 3 findings. Two are claims the fix took from the review.

1. **BLOCKING.** The text is card 3 (`03-pattern-shorten.md`), moved to come before every card that grows `PATTERN.md`.
   - The fix rests on the review's claim that "each intermediate size stays at or below the final projection that pattern-headroom already admits". That bound is not enough. Once card 3 lands, each later card's lint-pattern step compares its result with a HEAD that is already shortened. It passes only at or below `HeadroomBytes`, which is 15564. The base size, 15651, no longer counts.
   - Measured on the worktree, card 3 saves about 482 bytes, taking `PATTERN.md` from 15651 to about 15169. The growth after it is +69 (card 4), +122 (card 8), +133 (card 20) and +137 (card 32's `PATTERN-tier-tags` line).
   - Card 20's gate ends near 15493, which passes. Card 32's gate grows from 15493 to about 15630, which is above 15564, so it fails. F1's premise, a per-card lint-pattern failure, is still unresolved at card 32.
   - The fix needs about 66 more bytes cut so the final size is at or below 15564. Card 3's Intent sentence "stays under its base size" should then name the headroom line, not the base.
   - Evidence: `internal/pattern/headroom.go:12`, `if len(head) &lt;= len(base) || len(head) &lt;= HeadroomBytes`, with base being HEAD at the card gate.

2. **MEDIUM.** The text is card 6 (`06-landing-row-keys.md`): "the hand-built Deps … set `GateKey` … `merge-in` in … `TestVerifyGate_CheckMapsVerifyResult`".
   - That test builds no `Deps`. It builds a `verifyGate{...}` literal through `newGateFixture` (`internal/landingshed/verifygate_test.go:95-113`), and `gate = verifyGate{}` at line 451.
   - So the step as written cannot be carried out. The test's row key goes on the `verifyGate` field that card 6 adds, set in `newGateFixture`. `newGateFixture` is not listed under card 6's Edit.

3. **LOW.** The text is card 8 (`08-green-record.md`): "`TestGreenRecord` pins the rows `TestGateTest_Scenario` does not reach … the … `-- -count=1` carry breaks are left to the scenario".
   - The old row also pinned that an entry recorded under `-- -count=1` then carries on an exact repeat. That positive exact-args carry has been dropped from `TestGreenRecord`.
   - The scenario rows in card 8 cover only the break: "a changed tier command, a different tree and `-- -count=1` each break the carry". So "pins the rows the scenario does not reach" no longer holds for the exact-match carry.

Tool calls:
- Measured the byte saving of card 3's nineteen lines against `PATTERN.md` and the `PATTERN-tier-tags` line length. My context had the line texts but no sizes.
- Read which function owns `beginbatch_test.go:606`, and looked for `Deps{`/`verifyGate{` in `verifygate_test.go`. My context had only lines 207–250 of that test.
- Found where `TestVerifyGate_CheckMapsVerifyResult` builds its gate, and read `lyx config --help` for `--layer local`. Neither was in my context.
- Read `newGateFixture` (`verifygate_test.go:95-115`) to confirm the gate is built without `Deps`.
