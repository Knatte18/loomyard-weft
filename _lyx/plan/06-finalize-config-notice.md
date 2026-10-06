# Card 6 — finalize-config-notice

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps`
- `internal/landingshed/finalize_test.go`
- `internal/landingshed/finalize_integration_test.go`
- `internal/loomcli/landingdeps_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#ReadConfigChanges`
- `plan:internal/fabricengine#ConfigChanges`
- `internal/orchengine#QueueNotice`
- `internal/orchcli#PrimePaths`
- `internal/configreg#Modules`
- `internal/configengine#ConfigFileRel`
- `internal/fabricengine#PrimeName`

**Intent:**
Fixes `config-reaches-parent` (D4): Finalize reports a task's per-worktree config changes to the hub's orch notice queue, and never carries or writes them.

`landingshed.Deps` gains two told seams, both nil-is-absent like `MarkTaskDone`, documented in the same style and naming neither fabric side (`PATTERN-told-geometry`, `PATTERN-fabric-vocabulary`):
- `ConfigChanges func() (fabricengine.ConfigChanges, error)`: the task's changes to its own per-worktree config files since its fork point;
- `Notify func(line string) error`: queues one single-line notice for the hub's orchestrator.

`Finalize.Call` reads `ConfigChanges` as a new step right after the status commit (step 1b) and before the catch-up merge-in, so it runs before any parent-side mutation.
It composes at most one notice line:
- non-empty `Files`: the task branch, the changed files, the `Base` and `Tip` SHAs, and the instruction that landing does not carry them to the parent branch, so each change meant for the parent is re-applied on the parent's copy of the same file;
- a read error: a warning is logged and the line says the task branch's config changes could not be read, carries the error text (which names the two branches to compare), and asks for a diff by hand;
- empty `Files`: no line.
The line starts with `loom:`, is collapsed to one line (any newline in error text folded to a space), and spells no reserved directory literal (`PATTERN-lyxdirs-single-declarer`).
`pushParent` takes the line and, on its Done return only, calls `Notify`; a `Notify` error is a logged warning and never changes the verdict, and a nil `Notify` or empty line does nothing.
A Stuck or error outcome queues nothing.
Because an already-landed parent takes the same Done path, a Finalize resumed after its landing completed queues the notice again (R2), squash or not.
Bound: Finalize never writes the parent's config and never stops on this output; the notice is the only new output.

`landingDeps` fills both:
- `ConfigChanges` calls `fabricengine.ReadConfigChanges(l, taskBranch, parentBranch, rels)`, where `rels` is `configengine.ConfigFileRel(m.Name)` for every `configreg.Modules()` entry with `HubWide` false, so a HubWide module's worktree copy is never read;
- `Notify` resolves the prime's location at call time (`fabricengine.PrimeName(l)`, then `lyxcwd.ResolveWorktree` on `fabricengine.WorktreePath(l, primeName)`) and calls `orchengine.QueueNotice(orchcli.PrimePaths(prime), line, time.Now())`, returning only its error; `queued=false` for want of an orch strand is accepted, since `QueueNotice` already logs it (A1, A2).
The reflection drift guard in `landingdeps_test.go` covers the new fields as-is.

Docs: `internal/landingshed/doc.go`'s Finalize paragraph says Finalize reads the task's config changes before landing and queues one notice at Done, never carrying them.

Tests:
- `internal/landingshed/finalize_test.go` (fakes, untagged), one table-driven test over the seams: a non-empty diff queues exactly one notice at Done naming the task branch, each file, both SHAs and the re-apply instruction; an empty diff queues none; a read error still reaches Done and queues one notice saying the diff could not be read, carrying the error text; a `Notify` error leaves the verdict Done; a failed parent-side merge (Stuck) queues none; an already-up-to-date merge queues the notice again; nil seams change nothing.
- `internal/landingshed/finalize_integration_test.go`: the already-landed idempotent test wires `ConfigChanges` to `fabricengine.ReadConfigChanges` on the real pair after committing a `loom.yaml` change on the task side, and a capturing `Notify`; it asserts one notice naming the file, for the squash and the non-squash landing, and that the parent pair's config file is byte-identical afterwards.
- `internal/loomcli/landingdeps_integration_test.go`: on a `hubforge` hub with a pair whose task side committed changes to anchored `loom.yaml` and `board.yaml` config files, `landingDeps(...).ConfigChanges` reports `loom.yaml` and not the HubWide `board.yaml`; with an orch strand recorded in the prime's orch state (`orchengine.SaveState`), `Notify` queues one notice file in the prime's notices directory; with none recorded, `Notify` returns nil and queues nothing.

**ImpactSummary:** landingshed Deps and Finalize gain the config-change notice; loomcli's landingDeps wires it; landing verdicts and parent writes are unchanged.

**Commit:** 6: loom: Finalize reports a task's config changes to the hub orch
