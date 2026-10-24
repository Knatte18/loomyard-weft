# Card 2 — gate-flag

**Edit:**
- `internal/gatecli#newCommand`
- `internal/gatecli#testRequest`
- `internal/gatecli#flagShapeRefusal`
- `internal/gatecli#runTest`
- `internal/gatecli#computeScope`
- `internal/gatecli#listedTiers`
- `internal/gatecli#execution.report`
- `internal/gateslot#RunRecord`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestPlanTierRuns`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gateslot#TestWriteRunRecord`
- `internal/gatecli/doc.go`

**Intent:**
Add `lyx gate test --gate <key>` (D3), the call every site renders from card 3 on.

`newCommand` registers `--gate <key>`, and `testRequest` carries it.
`flagShapeRefusal` refuses `--gate` beside `--tier`, `--upto`, `--all` or `--base` before hub resolution, naming the key and the way forward of dropping the other flag, since the key's tiers and scope come from `gate.yaml`; `TestGateTest_RefusesBeforeResolvingTheHub` gains one row per flag.
Without `--gate` the verb keeps its manual flags as today.

Once the config is loaded, `runTest` reads the key through `Config.Gate`, so an unknown key refuses with that function's way forward naming `lyx config gate`.
The key's tiers are the listed tiers, through `listedTiers`, and its scope picks the scope form `computeScope` resolves:
`card` takes the `--card` ids as the card form and refuses a call with no `--card`, naming the key and the way forward of passing `--card <id>` per card; `all` is the `--all` form; `diff` is the computed form, the diff from the merge base with the parent branch, whole without a parent, until card 8 adds the green-record base.
`--card` ids beside a key whose scope is not `card` are ignored.
Explicit targets beside `--gate` replace the key's scope, which only darn's tree-scoped sites render (D14).
`TestPlanTierRuns` gains rows for a key's tiers, and `TestGateTest_Scopes` rows for a `card` scope without `--card` (the refusal names the key), a `card` scope with `--card`, an `all` scope, explicit targets beside `--gate`, `--gate nosuch` refused with `Config.Gate`'s way forward naming `lyx config gate`, and `--card 1` beside an `all`-scope key running the key's `all` scope.

`RunRecord` gains `GateKey` (`gate_key`, omitted when empty), the key a `--gate` call ran and empty for a manual run; `execution.report` fills it, and `TestWriteRunRecord` round-trips it.
The verb's long help and the package doc's Verb and Scope sections name `--gate`, its refusals and that explicit targets replace the key's scope.

Bound: `--gate` lets a caller run only the tiers and scope a hub-file key names; it opens no flag combination the manual flags lacked.

**ImpactSummary:** Adds one flag and its refusals to `lyx gate test` and one field to the run record; no existing call changes.
