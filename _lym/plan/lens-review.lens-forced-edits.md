```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. I checked what each card's change forces outside the members it names, and found three defects.

1. **BLOCKING — Card 7 (bouncer-review-inputs): the re-signed `ReviewInput` seam has callers no card lists.**
   Card 7 turns `ReviewInput func() (string, time.Time, bool, error)` into `ReviewInputs func() ([]ReviewInput, error)`, renaming it on `BouncerConfig` and `shedrecipe#Env`. These callers are left out:
   - `internal/loomcli#TestWire_DarnRecipe` (`internal/loomcli/wiring_darn_test.go:111-118`) checks `c.env.ReviewInput` against nil and calls it for four results.
   - `internal/loomcli#TestWire_ReviewKeysReachTheEnv` (`internal/loomcli/wiring_test.go:1014-1016`) checks `c.env.ReviewInput` against nil.
   - `internal/shedadapters#withReviewInput` (`internal/shedadapters/bouncer_amendment_test.go:81`) builds the seam with the old signature.
   - `internal/shedadapters#amendmentInputRow` (`bouncer_amendment_test.go:196`) reads the generation's `ReviewInput`.
   - `internal/shedadapters#reentryInputRow` (`internal/shedadapters/bouncer_reentry_test.go:182`) asserts `generation.ReviewInput != inputPath`, which card 7's `ReviewInputs []string` record changes.

   As planned, the card does not compile under its own gate. Fix: add all five glyphs under card 7's `**Edit:**`.

2. **MEDIUM — Card 7: `doc.go` is listed under `Uses` but its text becomes wrong.**
   Card 7 changes the seam's name, its shape and the full-generation rule. `internal/shedadapters/doc.go:293` and `:306` describe `ReviewInput`, and they would be left describing the old seam. Fix: move `//internal/shedadapters/doc.go` from `**Uses:**` to `**Edit:**` on card 7.

3. **LOW — Card 7: `envkit`'s `nilLegal` entry is a string key that must follow the rename.**
   `internal/testkit/envkit/envkit.go:145` has `"ReviewInput": true`. Card 7 lists `internal/testkit/envkit#nilLegal`, but its Intent never says the key becomes `"ReviewInputs"`. Fix: state the key rename in card 7's Intent so `NilSeams` keeps skipping the new field.

Tool calls beyond my context:
- Grepped every `ReviewInput` user and resolved card 7, 8 and 11 glyphs, because the exploration never listed the seam's test callers.
- Read the `ReviewInput` users in `wiring_darn_test.go`, `wiring_test.go`, `bouncer_reentry_test.go` and `envkit.go`, to see how each one breaks.
- Found the functions enclosing each user and searched the `pr-continue` listings, `PullRequestDeps` literals and `PR-Review` recipe tests, to get member glyphs and find verb and deps registration sites.
- Checked `cmd/lym/ghguard_test.go`, the `sampler-done` listings and `shellguard`, to see whether a new loom verb must be registered in a guard or help list.
- Grepped `loomcli` doc, `shedrecipe` doc and `docs/overview.md` for review-input, drift-judge and plain-include text, to check for forced documentation edits.
