```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1's cards 3, 4, 5, 14 and 20; F2's `tagged.go`, `allow_tmux.go` and follow-up items; F3; F4).

1. LOW: card 28's retag of `internal/planglyph/testdata/glyphchain/shapes/tagged.go` from `integration` to `tier1`, plus the new sentence in card 28's Intent.
   Summary: the retag removes the only test that pins planglyph's type load seeing an old-spelled file, before D13's follow-up drops that spelling. Listing the fixture's retag in card 31's follow-up entry would fix the same breakage and keep the pin.
   Evidence:
   - Card 11 widens `typesLoader.load` to `tier0,tier1,tier2,tier3,integration,tmux,llm`, so that "a branch not yet holding the retag still has its old-tagged files seen until D13's follow-up drops the old spellings".
   - In card 11, only impactset gains a test row (`TestDerive_OverARepository`, tier0). Planglyph's load is pinned only through `glyphchain_test.go:95,211-216` (`taggedFile = "shapes/tagged.go"`, the `shapes#Tagged` leg).
   - Once card 28 respells the fixture `tier1`, that leg proves the `tier1` term. If the `integration` term were dropped from `typesLoader.load` during the transition, no test would fail, though D5/D13 promise that term.
   - The review's premise only needs the fixture respelled when the follow-up drops `integration` from the load. Naming `tagged.go`'s retag in card 31's follow-up entry resolves it without losing the pin.

No other findings:
- Card 3's new readers: the scope-ignoring `Render` stated for `roundGateFixture.round` mirrors `gateListSource` and stops scope words reaching `GateCall` as card ids. The key each rendered call names matches card 1's renames (`finalize` for the landing sources, `code-burler-fix` for the fix rows' round source).
- Card 4's `intentionallyNil`: the field goes in the same card.
- Card 5's `TestVerifyGate_RealVerifytreeScenario`: it assigns `deps.VerifyCommand` directly, as cited.
- Card 14: `TestDerive_OverARepository`'s guard fixture `g/g_test.go` is untagged, so it reads as Tier 0.
- Card 20: `engineMinimalConfig`'s `Command` entry keeps `command`, which still builds. The doc paragraphs are prose-only.
- F2's `allow_tmux.go`: keeping `tmux || llm` beside `tier2 || tier3` is justified. `cmd/lyx/tmuxisolation_test.go:33` `isolationTagSets` compiles packages under `{"tmux"}` and `{"llm"}` alone, so a tier-only constraint would drop `tmuxAllowed` in those binaries until the follow-up.
- F3: the step can force through `--open-terminal` by wiring a `battenCLI{openTerminalFlag: true}`, as `wire_seams_test.go:259-264` already does, so the extension is feasible.
- F4: Edit is the right group for a rewritten line.

Tool calls:
- Grepped for `allow_tmux` and `isolationTags` consumers and for `integration`/`Tagged` in `glyphchain_test.go`: context lacked who reads the respelled constraints.
- Read `stepCommandRows_FollowTheTasksLocalSwitch`, `wire.go`'s force branch and `wire_seams_test.go`'s forced-answer block: context lacked how the step and the wiring force Open-Window.
- Read card 11 and the `tagged.go` readers in planglyph: context lacked which tests pin planglyph's load tags.
