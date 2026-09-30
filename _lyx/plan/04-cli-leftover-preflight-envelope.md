# Card 4 — cli-leftover-preflight-envelope

**Edit:**
- `internal/fabriccli/fabric.go`

**Create:**
- `internal/fabriccli/addleftover_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#ErrRemoteLeftover`
- `internal/fabriccli#errWithRecord`

**Intent:**
Surface Add's leftover-branch refusal as the pre-flight failure it is, per the Mutation Record Invariant: a bare `output.Err`, carrying neither `mutations` nor `partial`.
In `runAdd`, before the existing `errWithRecord` call, `errors.As` the error against `*fabricengine.ErrRemoteLeftover` and return `output.Err(out, err.Error())` on a match; every other Add error keeps its current envelope.

`addleftover_integration_test.go` (integration tag, package `fabriccli_test`, hub via `hubforge.NewHub`, driven through `fabriccli.RunCLIIn`): `add`, then plain `remove`, then push a commit onto origin's `<slug>-weft` from a clone of the hub's weft bare, then `add` again → non-zero exit, `"ok": false`, an `error` naming `<slug>-weft`, and no `mutations` or `partial` key in the envelope.

**ImpactSummary:** `runAdd` only; the envelope of every other Add failure is unchanged.
