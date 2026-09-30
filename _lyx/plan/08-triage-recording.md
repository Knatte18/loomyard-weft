# Card 8 — triage-recording

**Edit:**
- `triage.go`
- `triage_test.go`

**Create:**
- `internal/websterengine#triageWarnings`
- `internal/websterengine#triageStuckReason`
- `internal/websterengine#writeTriageFrictionNote`

**Uses:**
- `internal/websterengine#IntegrationTriage`
- `internal/websterengine#IntegrationFailure`
- `//internal/friction/friction.go`

**Intent:** Add the pure formatting and note-writing pieces the stage records a verdict with, so card 9 only wires them (D2, D11).

`triageWarnings` returns one warning per non-empty non-regression category, flaky then pre-existing, each naming its identities;
it returns nil for a triage with neither.

`triageStuckReason` builds the `RunResult.StuckReason` for a demoted run: it names every regressing identity and the localized card, or states the card was not localized when `offendingCard` is `unknown`.

`writeTriageFrictionNote` is a no-op when `frictionDir` is empty or both non-regression lists are empty.
Otherwise it writes a markdown note to `friction.NotePath(frictionDir, "webster-verify-triage")`, creating the directory with `friction.EnsureDir`:
a title line naming the integration-suite triage, then the flaky and pre-existing identities, each list under its own line of prose.
A pre-existing failure is environment trouble the hub collects from friction notes, which is why this note exists.
`websterengine` already imports `internal/friction`, so the Friction Leaf Invariant holds.

The untagged `triage_test.go` gains cases asserting:
the exact warnings for flaky-only, pre-existing-only, both, and neither;
the stuck reason naming two regressing identities and a card, and the not-localized wording for `unknown`;
the note's exact content under a `t.TempDir()` friction dir;
no file written when the dir is empty;
and no file written for a regression-only triage.

**ImpactSummary:** Adds three unexported helpers to `triage.go`; nothing calls them until card 9.

**Commit:** 8: triage-recording
