# Card 13 — fabricengine-add-dedupe

**Edit:**
- `fabricengine/add_test.go`

**Uses:**
- `plan:internal/fabricengine#ErrInvalidSlug`
- `fabricengine/slug_test.go`

**Intent:**
Stop `add_test.go` replaying `slug_test.go`'s rejection classes through `Add` (#325 §5); `slug_test.go` owns the classes.

Keep one `Add` test proving the wiring, asserting with `errors.Is(err, ErrInvalidSlug)`: one bad slug, one `cfg.Dirs()` junction name refused, the `_raddle` name accepted, and `.lyx` refused.
Delete the other `Add` slug-replay tests.

Prove the wiring test still bites: delete the `validateWorktreeSlug` call in `Add`, confirm a test in this file fails, and restore the call.

List every deleted leaf test in the commit body for the test-count accounting.
Apply D10 to the kept test.

**ImpactSummary:** One untagged fabricengine test file shrinks to a single Add wiring test; slug_test.go keeps every rejection class.
