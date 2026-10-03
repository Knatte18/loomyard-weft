# Card 36 — battencli-wiring-reflect

**Edit:**
- `battencli/wire_test.go`

**Create:**
- `battencli/wire_seams_test.go`

**Uses:**
- `internal/battencli#battenCLI.wire`
- `testkit/envkit/envkit.go`

**Intent:**
Give battencli the same reflective wiring test loomcli got in card 35 (D8, #324 §5).

`wire_seams_test.go` (untagged) holds `TestWire_EverySeamFilled`: it calls `wire()` with the synthetic location and absent task worktree `TestWire_SeedChildClosuresFilled` uses today, then requires `envkit.NilSeams(c.env)` to equal an `intentionallyNil` set exactly.
batten wires only its own producers, so its set is larger than loomcli's; every entry still names one field path with a one-line reason, one entry per line without aligned values (D10).

It replaces `TestWire_SeedChildClosuresFilled`, whose `Env.SeedChild` nil checks `NilSeams` covers.
`TestWire_CommitStatusFilled` stays: it checks `c.shedPaths.CommitStatus`, a `shedbuild.ShedPaths` field outside `shedrecipe.Env` that the reflective walk never sees.
Tests asserting behaviour stay too: `TestWire_LazySeams`, `TestWire_ChildDriverDelegatesToChildDriverOf`, the refusal and teardown tests, and the rest of `wire_test.go`.

Prove the test: add an unset func field to `shedrecipe.Env`, watch `TestWire_EverySeamFilled` fail naming it, then remove the field.

What the escape hatch lets through: a field in `intentionallyNil` stays nil unchecked, bounded by its per-entry reason and review.
List the deleted leaf tests in the commit body for the test-count accounting.

**ImpactSummary:** battencli wiring tests only; the SeedChild nil-check test becomes one reflective test, while the CommitStatus check outside Env and every behaviour test are kept.
