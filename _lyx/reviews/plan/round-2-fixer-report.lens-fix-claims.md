```yaml
lens: fix-claims
outcome: reported
```

Findings:

1. MEDIUM — card 28's new sentence: "the card keeps the `tmuxkit` package glyph, since a file glyph beside it fails `redundant-file-target`."
   Summary: this states a rule the code does not hold. The fix took the claim over from the reviewer.
   Evidence: `internal/planglyph/doc.go:77` and `contracts/specs/loom-plan-spec.md:498` define `redundant-file-target` as "a card's own target list holds a file self glyph beside a member glyph that resolves into that file".
   A package self glyph such as `internal/testkit/tmuxkit#` is not a member glyph, so a file glyph for `allow_tmux.go` beside it would not raise that finding.
   The review's own Rejected section says `redundant-package-target` "fires only beside a member of the same package".
   The rationale should be dropped, or restated as a choice: the package glyph already covers the file.

2. MEDIUM — card 5's rewritten sentence: "`envkit.LandingDeps`, landingshed's `newGateScratch` and the `integration`-tagged `TestVerifyGate_RealVerifytreeScenario` … set a keyed closure".
   Summary: `envkit.LandingDeps` does not set `VerifyCommand`, so there is no closure to re-key there. The claim comes from the original card text, but it sits in the sentence the fix rewrote.
   Evidence: `internal/testkit/envkit/envkit.go:148-166`, the body of `LandingDeps`, has no `VerifyCommand` field.
   `envkit.go:70` lists `"Landing.VerifyCommand": true` among the nil-by-default seams.
   The `VerifyCommand` closure at `envkit.go:258` belongs to the shedrecipe `Env` (`FullEnv`), not to `landingshed.Deps`.
   Either drop `envkit.LandingDeps` from the "set a keyed closure" list, or say it starts setting one.

The other claims I checked hold:
- Card 3's `--gate` expectations: `TestWire_DarnRecipe` asserts `VerifyCommand` at lines 104 and 141, and both tests assert `Round().Render`.
- Card 4's `intentionallyNil` row.
- Card 5's integration tag (line 1 of the file).
- Card 14's `Tier: 0`: the guard fixture `g/g_test.go` is untagged.
- Card 20's `open_window: false` fixture.
- Card 21's doc line, which is `loomrecipe/doc.go:23`.
- The overview's and card 23's statement that respelled support files compile under the same tag sets: the Constraints pair each `tierN` with its old spelling in every command and vet step.

Tool calls:
- Read `envkit.LandingDeps` and the `NilSeams` list, because my context showed only `envkit.go:163` and `:258`, not the `LandingDeps` body.
- Read how `darntreescope_integration_test.go` builds the landing `Deps`, to confirm `landingCall` comes from darn's landing source.
- Read the definition of `redundant-file-target`, which my context did not hold, to test card 28's rationale.
