# Card 20 — spawn-role-constants

**Edit:**
- `internal/loomcli/driverspec.go`
- `internal/loomengine/discussion.go`
- `internal/loomengine/plan.go`
- `internal/loomengine/rework.go`
- `internal/burlerengine/engine.go`
- `internal/frictionengine/spec.go`
- `internal/landingshed/describe.go`
- `internal/mergeresolve/spec.go`
- `internal/treadleengine/judge.go`
- `internal/treadleengine/targeting.go`
- `PATTERN.md`
- `pattern/PATTERN-agent-name.md`

**Create:**
- `cmd/lyx/spawnrole_test.go`

**Intent:** Every spawn site takes its agent role from a named constant declared in its own module (D12).

Replace each inline role literal with an unexported (or, where another package of the same module must name it, exported) constant declared once in the spawning package.
The literals are:
- `"driver"` in `loomcli`;
- `"discussion"`, `"plan"` and `"rework"` in `loomengine`;
- `"burler"` in `burlerengine`;
- `"friction"` in `frictionengine`;
- `"describe"` in `landingshed`;
- `"conflict"` in `mergeresolve`;
- `"judge"`, `"triage"` and `"targeting"` in `treadleengine`.

Each constant's doc comment says it is the agent-name role this module's spawn carries.
Values do not change, so agent names, run directories and every `agentname.Matches` query keep resolving.
Grep each package's tests for the literal role strings, and switch any test that asserts the spawned spec's role to the constant where it is in the same package.
Tests that assert the on-the-wire name keep the literal.
There is no shared list, and `agentname`'s role set stays open.

`cmd/lyx/spawnrole_test.go` is an untagged AST scan over every non-test `.go` file in the module.
It fails any composite literal of `shuttleengine.Spec` (and of reed's strand-spawn spec type, whatever struct `reedengine`/`reedcli` take a `Role` in) whose `Role` field is a string literal.
A constant, a variable or a parameter passes.
It is a tripwire, not a completeness proof.
Its failure message names the file, the line and `PATTERN-agent-name`.
A unit case runs the scan over an in-memory fixture source with an inline literal and asserts the finding.
The scan sits beside the other `cmd/lyx` enforcement scans and reuses `internal/testkit/scankit`.

`PATTERN-agent-name`'s line and background state that the spawning module owns its role names as constants, with no shared list, and name the new scan.

**ImpactSummary:** Literal-to-constant swaps at each spawn site with unchanged values, plus one new enforcement scan; no runtime behavior changes.
