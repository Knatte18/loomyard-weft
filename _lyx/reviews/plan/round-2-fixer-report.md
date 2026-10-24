# Plan round 2 — fixer report

Every finding's premise held against the code at HEAD.

## Changes

- **F1 (BLOCKING), fixed as proposed, plus readers found by rerunning its searches.**
  - Card 3: the Edit group lists the ten readers the review named and `contracts/specs/loom-status-spec.md`.
    The status spec was found by a search for `GateCall(` and the old `gate test --wait --tier` literal over the module's tests and docs; its running-verify example rendered the old call.
    The Intent names the search and gives the `--gate` spellings each reader asserts: `--gate card --card <id>` and `--gate batch` with one `--card` per card.
    `wantDarnLandingCall` and `landingCall` become `--gate finalize`.
    The `Round().Render` assertions become `--gate code-burler-fix`, the key card 1 gives the fix rows' round source.
    `roundGateFixture.round`'s `Render` ignores scope words until card 4 deletes it.
  - Card 4: lists `internal/battencli#intentionallyNil`, whose `Round` row goes with the field, and its ImpactSummary names battencli's seam list.
  - Card 5: lists `TestVerifyGate_RealVerifytreeScenario` as a test that sets `deps.VerifyCommand` to a keyed closure, found by a search for `VerifyCommand` over the landing tests and envkit.
  - Card 14: lists `TestDerive_OverARepository`.
    Its Intent says the two `Guards` assertions compare `GuardTest` values with their tier (`{Name: "TestGuard", Tier: 0}`, since the fixture's guard file is untagged).
  - Card 20: lists `internal/shedbuild#engineMinimalConfig`, `internal/battenshed/doc.go` and `docs/overview.md`.
    Its Intent says the fixture drops the `switch` key and the docs' Command rows paragraphs describe the entry gate, and its ImpactSummary names both.
    A search for `"switch"`, `SwitchModule`, `SwitchKey` and `Forced` found no other unlisted reader; `battenCLI.reopenWindow`'s `CommandRow` literal is already on the card.
- **F2 (MEDIUM), fixed differently in part.**
  - Proposed `!tier1` for `TestMain` companions: done, but generalized.
    The proposal's premise, that the `integration` sibling becomes `tier1` after the retag, holds only for siblings that hold tests (`gitkit`, `logger`).
    Under the old rule, a support-only sibling such as `internal/battencli/testmain_integration_test.go` keeps `integration`.
    With companions at `!tier1` and siblings left at `integration`, a tier1 binary would hold no `TestMain` once the follow-up drops `integration`.
    A search found forty-odd support-only test files constrained `integration`, `tmux || llm`, `integration || tmux` or `tmux && !windows`, all of which break the same way.
    So the overview's retag-rules decision and card 23's rule paragraph now say a support-only file keeps its constraint's shape with each old tag spelled as its tier tag: `!integration` becomes `!tier1` (with the companion's comment naming `tier1`), `integration` becomes `tier1`, and `tmux || llm` becomes `tier2 || tier3`.
    The decision states why this is safe: every transitional tier command and vet step pairs `tierN` with its old spelling, so each respelled file compiles under exactly the commands it did before.
  - `allow_tmux.go`: fixed as proposed in card 28 (`tier2 || tier3 || tmux || llm`, comments naming the tier tags, package glyph kept), and card 31's follow-up removes its `tmux` and `llm` terms.
  - `tagged.go`: fixed differently.
    The proposal respells it `tier1` in card 28.
    Following the self-check's fix-proposal finding, card 28 instead leaves it at `integration`, where it keeps pinning that the type load card 11 widens sees an old spelling.
    Card 31's follow-up entry respells it when it drops `integration` from that load.
  - Card 28's ImpactSummary names `allow_tmux.go`.
- **F3 (MEDIUM), fixed as proposed.**
  Card 20's Create group and the `TestWireSetting_OpenTerminalForcesOpenWindow` Intent paragraph are gone.
  `stepCommandRows_FollowTheTasksLocalSwitch` now asserts, over its `open_window: false` task worktree, that `Env.Setting` answers `batten.open_window` false without `--open-terminal` and true with it, and that `seed_ide` is unchanged.
- **F4 (LOW), fixed as proposed.**
  `internal/loomrecipe/doc.go` moved from card 21's Extends group, which is dropped, to its Edit group.
  The Intent says the budget-0 routing line is rewritten.

## Self-check

- fix-consistency, LOW: card 4's Intent left the `Round().Render` assertions unstated. Fixed as proposed: card 4 says `TestWire_DarnRecipe` and `TestWire_DefaultConfig` drop them with the field.
- fix-consistency, LOW: card 23's testdata statement disagreed with card 28's `tagged.go` exception. Fixed with own change: the exception is gone, since `tagged.go` now stays at `integration` (see the fix-proposal line below).
- fix-claims, MEDIUM: card 28's `redundant-file-target` rationale was false (`internal/planglyph/doc.go:77`: the rule fires beside a member glyph, not a package glyph). Fixed as proposed: card 28 says the package glyph already covers the file.
- fix-claims, MEDIUM: card 5 said `envkit.LandingDeps` sets a keyed `VerifyCommand` closure, but `LandingDeps` sets no `VerifyCommand` (`envkit.go:148-166`). Fixed as proposed: it is dropped from that list; its `GateKey` line stays.
- fix-proposal, LOW: respelling `tagged.go` in card 28 loses the pin on the type load's old spelling. Fixed as proposed: the respelling moves to card 31's follow-up entry.
- fix-affected-set, LOW: the crucible docs tell sessions to run `go test -tags tmux` and `-tags llm`. Fixed with own change: the fork proposed adding them to the follow-up, but they stop compiling the tests once the retag lands, not when the follow-up does. Card 31, the last retag card, lists `crucible/README.md`, `crucible/orchestrator-prompt.md` and `crucible/review-prompt-template.md` and respells them `-tags tier2` and `-tags tier3`.

## Disputed

None.

## Deferred

None.
