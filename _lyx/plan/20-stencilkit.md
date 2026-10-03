# Card 20 — stencilkit

**Create:**
- `internal/testkit/stencilkit/stencilkit.go`
- `internal/testkit/stencilkit/stencilkit_test.go`

**Edit:**
- `docs/overview.md`

**Uses:**
- `contracts/stencils/stencils.go`
- `internal/stencilstore/reconcile.go`

**Intent:** Add `internal/testkit/stencilkit`, which seeds stencil fixtures from `stencils.Registry()` through `stencilstore.Reconcile`, the owner the Stencil Ownership Invariant names (D11).
It offers a seed of a fresh `t.TempDir()`, a seed into a caller-given directory (a fixture hub's stencils directory), and a removal of named stencils for deliberate-absence tests ("seed all, then remove X").
Its only assertions are `t.Fatalf` on its own setup.
Seeded files carry `Reconcile`'s hash stamp, as production's do, where the hand seeders wrote unstamped bytes.
`stencilkit_test.go` proves that a seeded directory reads back every registry name through `stencilstore.Read` with a body equal to the registry default once the stamp is stripped (`stencil.StripLeadingComment`), and that a removed name is absent while the rest stay.
`docs/overview.md`'s Tests passage names `stencilkit`.
The packages `stencilkit` imports (`contracts/stencils`, `internal/stencilstore` and their own imports) cannot use it from their internal tests without a cycle;
none of them hand-lists stencils today.

**ImpactSummary:** One overview sentence alongside the new kit; no existing caller changes in this card.
