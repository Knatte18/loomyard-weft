# Card gate runs own packages only; guard step and dependents at batch

The per-card gate (`lym gate test --gate card --card <id>`) used to run every package that depends on the card's packages,
then a guard step over the repository's `//lym:guard` tests.
The batch gate after the batch's last card runs both again.
`docs/research/webster/gate-time-2026-11-28.md` measured the card gate's listed tier0 run at 44.8 s per run over 75.5 packages,
of which 2.3 are the card's own, against 5.5 s over 3.3 packages before the dependent scope landed.

## What changes

- **`--gate card` runs the cards' packages alone.**
  Every listed tier of the `card` key runs the cards' packages, plus the main packages under `--main-packages`.
  The dependent packages are not listed at all, so the call no longer runs `go list` to find them.
  The call prints one stderr line saying so and naming `--gate batch` as the call that runs the dependents and the guard step.
- **`--gate card` plans no guard step.**
  `--guards` still forces one.
  The vet step over the cards' packages stays.
- A `--card` call under the `batch` key or under no key is unchanged.
- **The card gate's cold fixer re-runs only the checks that failed.**
  `contracts/stencils/gatefix/gatefix-body-card.md` used to have the fixer re-run the failed check and then the whole card gate call.
  The detached runner already runs that call again over the fixer's tree once the fixer ends, and that run's verdict is the one the implementer reads.
  The prompt now has the fixer re-run each failed test through `lym gate test --tier <tier> <package> -- -run '^(<name>)$'`,
  and any other failed step by the command the report names.
  It still names the whole call, as the one that judges the fix, and tells the fixer never to run it.
- `gateslot.CardGateKey` replaces the unexported `cardGateKey`, the duplicate `batchGateKey` in `priority.go`,
  Weaver's `gateKeyCard` and the `"card"` literals in `runplan` and `webstercli`.
- The Webster implementer stencil moves its guard-test fix rule from the card gate step to the batch gate section,
  where such a failure now surfaces, and the fix lands as a ` (fix)` commit.
  The Weaver batch template says what the batch gate adds over the card gates.
- The plan spec's verify model, the `gatecli` package doc, the `--card` flag help, the `gate.yaml` template comment
  and `PATTERN-test-speed` state the new split.

## Design decisions

- **The guard step moves to the batch gate.**
  The research measured the card gate's guard step at 8.1 of 28.7 card gate minutes per implementer row.
  It estimated that narrowing the listed run to the card's own packages adds about 4.4 min of guard tests back,
  since the narrow run covers fewer of them.
  Against that, 11 batches in its sample had a failed card guard step,
  each of which would now cost one more batch gate round.
  The research put such a round at 3.0 min with the batch gate's tier1 over the dependent packages, and about 1.2 min once that tier runs the cards' packages alone, which landed before this change.
  On those figures the guard step costs the card gates about 12.5 min per row,
  and moving it costs about 2.2 min per row of extra batch rounds, so the trade still holds.
  These are estimates from earlier runs' figures: no row has run under the new scope.
- The rule is keyed to the `card` gate key in code, as the batch key's narrowing is, rather than a new `gate.yaml` field,
  so it takes effect without a hub config write.
- A `--card` call with no key keeps the dependent scope and the guard step,
  so a hand run of `--card` still reaches everything the card can break.
- The fixer prompt keeps the `gate_call` marker, naming the call it must not run, so the marker set and its test stay as they are.

## Tests

- `TestGateTest_Scopes` (`internal/gatecli/scopes_integration_test.go`):
  the `--gate card` row now asserts the root and nested modules' own packages alone, no guard run, and the stderr line;
  the batch-versus-card row asserts `--gate card` runs both listed tiers over `./lib` and `.` with no guard step;
  the held-test row that checks a held guard stays out of the guard step now runs `--gate batch`, the key that still has one.

## To check by hand

- A failure that only a dependent package's test or a guard test catches now surfaces at the batch gate.
  The batch key has no cold fixer, so the implementer fixes it under its self-fix bound.
- No test pins the fixer prompt's new rule; read `contracts/stencils/gatefix/gatefix-body-card.md`.

## Merge danger

**Door:** two-way
**Blast radius:** gate
