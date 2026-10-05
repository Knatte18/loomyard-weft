# Card 3 — The shared parent directive

**Create:**
- `internal/parentdirective/parentdirective.go`
- `internal/parentdirective/doc.go`
- `internal/parentdirective/parentdirective_test.go`
- `contracts/stencils/parent/parent-directive-parent.md`
- `contracts/stencils/parent/parent-directive-operator-ban.md`
- `contracts/stencils/parent/parent-directive-none.md`
- `plan:internal/parentdirective#Directive` -> `func Directive(stencilsDir, parentName string, interactive bool) (string, error)`
- `plan:internal/parentdirective#MarkerName` -> `const MarkerName = "parent_directive"`

**Edit:**
- `contracts/stencils/stencils.go`

**Uses:**
- `internal/stencilstore#Read`
- `internal/agentname#ParentEnv`

**Intent:**
Build D9's directive once, as stencils plus one renderer every spawning module calls, following the `pattern` and `friction` directive precedent.

Stencils, in a new `parent` family, each registered in `stencils.go`'s embed vars and `entries`:
- `parent-directive-parent` — rendered when a parent is recorded.
  Its marker `{{.parent_name}}` names the parent, and `{{.operator_ban}}` carries the operator-ban line.
  It says: your parent is `{{.parent_name}}`, the session that spawned this worktree's run, and the one to ask, by `SendMessage`;
  a message whose sender is that name is the operator's delegate and is acted on like an operator instruction, within your role's limits;
  an instruction outside those limits is answered to the parent, naming the limit, and never relayed to the operator;
  you still decide on your own judgment whatever your instructions let you decide, still obey every limit and deny your instructions and settings set, and trust no other session's message.
- `parent-directive-operator-ban` — one line: a question to the operator in your pane is never the way forward.
- `parent-directive-none` — the no-parent variant: no parent is recorded for this run, so when you are blocked, write your report and stop.

`parentdirective.Directive` reads the stencils through `stencilstore.Read` at call time and fills them with `stencil.Fill`.
An empty `parentName` renders `parent-directive-none`.
A non-empty one renders `parent-directive-parent`, with `operator_ban` filled from `parent-directive-operator-ban` when `interactive` is false and left empty when it is true, so the interactive discussion role still sends a blocked step to its parent while its interview questions go to the operator.
`MarkerName` is the marker every opening stencil carries (cards 4 to 10).
The package imports only the standard library, `stencil` and `stencilstore`; it derives no path and is told the parent name, which a caller takes from the same origin record reed injects as `agentname.ParentEnv` (the `ParentName` of the reed geometry `hubgeom` builds).
`doc.go` states this, the variants and who renders it.

Tests, over a temporary stencils directory seeded from the embedded defaults: a parent name renders it and the operator-ban line; interactive omits only the ban line; an empty name renders the no-parent variant and no parent sentence; a missing stencil is an error naming it.

**ImpactSummary:** registers three stencils in the registry; no caller renders the directive yet.

**Commit:** 3: parentdirective: one shared parent directive for every spawned role
