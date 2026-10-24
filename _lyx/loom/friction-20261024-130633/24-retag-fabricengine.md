# Retagging fabricengine, loom, reed and shuttle test files

The card gate (`--tier 0`) does not compile a package under the tier1, tier2 and tier3 tag sets, so a helper that a retagged tier0 file defines and a tier1 or tier2 file uses fails only in the batch gate's vet steps.
Each card gate passed while the batch gate's vet found about twenty such helpers across the cards; they were moved into new untagged `*_helper_test.go` files, and `shuttleengine`'s two untagged fake and fixture files were tagged `tier0` instead, since only tier0 tests use them.
A card's gate step that also vetted the card's packages under the other tag sets would have shown the helper dependencies up front.

`TestBuildSettings_MatchesGolden` in `internal/shuttleengine/claudeengine` fails at HEAD independent of this batch: card 12 (`91f8c5d5c`) added `tier2|tier3` to the shell guard's tags regex and `testdata/settings-golden.json` still carries `(tmux|llm)`.
The golden is outside this batch's cards, so it is left to the final gate's fixer.
