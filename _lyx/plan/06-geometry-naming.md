# Card 6 — geometry-naming

**Edit:**
- `internal/reedengine/geometry.go`
- `internal/hubgeom/hubgeom.go`
- `internal/hubgeom/doc.go`
- `internal/standalonegeom/reedgeom.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `internal/burlercli/wiring.go`
- `internal/loomcli/wiring.go`
- `internal/shuttlecli/cli.go`
- `internal/orchcli/cli.go`
- `internal/battencli/wire.go`
- `internal/webstercli/wiring.go`
- `internal/reedcli/watchdog.go`
- `internal/reedcli/cli.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/treadleengine/smoke_judge_test.go`
- `internal/loomcli/smoke_gate_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`

**Create:**
- `internal/hubgeom/reedgeom_integration_test.go`

**Uses:**
- `internal/agentname/agentname.go`
- `internal/fabricengine/worktreelist.go`
- `internal/shedrun/seed.go`

**Intent:**
Tell reed the name prefix and parent through its geometry; reed resolves neither itself (Told-Geometry Invariant).
Nothing in reed reads the new fields yet, so this card changes no behaviour.

`reedengine.Geometry` gains three fields, each documented:
`NameCode` (the code every strand name starts with; empty means the hub records none),
`NameSlug` (the task worktree's slug segment; empty in the prime and in a standalone run),
and `ParentName` (the full name of the session that spawned this worktree's default run; empty when none is recorded).
`ParentName` is unrelated to `Strand.Parent`, which stays a layout guid; the field doc says so.

`hubgeom.ReedGeometry` becomes `ReedGeometry(l *lyxcwd.Location) (reedengine.Geometry, error)`.
It reads `fabricengine.ReadCode(fabricengine.BoardDir(l.HubPath))` into `NameCode` (absent leaves it empty; reed refuses the spawn later, not here, so `reed status`, `down` and the watchdog keep working on an uncoded hub).
It compares `l.WorktreeName` against `fabricengine.PrimeName(l)`: the prime leaves `NameSlug` and `ParentName` empty;
a task worktree sets `NameSlug` to its raw worktree name (reed validates it at spawn) and reads its default run's seed through `shedrun.ReadSeed(l, shedrun.SelfRunID)` — never a joined `self` path — into `ParentName`.
A `PrimeName` failure is the function's only error;
an unreadable seed logs a named `logger.Warn` and leaves `ParentName` empty, since the parent is an optional escalation channel and must not stop `reed down`.
The doc comment and `hubgeom/doc.go` drop the "performs no I/O" claim and state what it now reads and why it is still the only reader.

`standalonegeom.ReedGeometry` sets `NameCode` to `agentname.StandaloneCode(hash8)` and leaves `NameSlug` and `ParentName` empty: a standalone run has no weft to record a code on and no seed.

Every production caller of `hubgeom.ReedGeometry` handles the error on the path it already uses for a failing geometry or config load (an `output.Err` envelope in a pre-run, a returned error in a wiring function or closure);
the smoke- and integration-tagged test callers listed above fail the test on it.

Tests: `standalonegeom_test.go` asserts the derived code and that it passes `agentname.ValidateCode`.
`reedgeom_integration_test.go` (`integration` tag) on a `hubforge` hub: the prime's geometry carries the fixture code with empty slug and parent;
a task worktree added with `lyx fabric add` carries its slug, an empty parent before seeding, and after `shedrun.WriteSeed(l, shedrun.SelfRunID, …)` with a `Parent`, that parent;
a hub whose `.lyx-code` is absent yields an empty code and no error.

**ImpactSummary:** Changes hubgeom.ReedGeometry's signature for every CLI wiring caller and the tagged tests that build a hub-mode reed engine, and adds three Geometry fields nothing reads yet.

**Commit:** 6: hubgeom: tell reed the name prefix and parent
