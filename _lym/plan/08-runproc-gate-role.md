# Card 8 — runproc-gate-role

**Create:**
- `plan:internal/runproc#RoleGate` -> `const RoleGate = "gate"`

**Extends:**
- `//internal/runproc/doc.go`

**Intent:** D3's gate role for the owner record: a detached gate runner registers under its task's slug with `RoleGate`, so pair teardown's `runproc.EndSparing`, which spares only `RoleDobby`, and a session end that ends the slug's processes end it (PATTERN-run-process-owner).
The constant's comment says it is a detached `lym gate` runner of a capped gate call.
The doc's list of roles gains `RoleGate`.
A role name carries no behavior of its own; card 11's registration row pins its use.

**ImpactSummary:** one constant added to runproc's role names and its doc list; no end sequence changes.
