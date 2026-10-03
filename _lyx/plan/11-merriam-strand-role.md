# Card 11 — Merriam's strand role is `webster`

**Edit:**
- `internal/websterengine/roles.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`

**Intent:**
D13: `roles.go` declares a strand-role constant `webster`, separate from `RoleMaster`, which stays the `webster.yaml` model key `master`.
`Run` spawns Merriam with `Spec.Role` set to the new constant, so reed names the strand `<shortname>:<slug>:webster`, while the model still comes from `deps.Roles[RoleMaster]`.
Existing `webster.yaml` files load unchanged.

Test in `runlevel_test.go`: the spec the fake starter receives carries role `webster` and the model resolved for `RoleMaster`.

**ImpactSummary:** Merriam's reed strand name changes from `…:master` to `…:webster`; config and model resolution are unchanged.
