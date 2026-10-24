---
verdict: APPROVED
---

No findings.
The five focus targets each settle as "not a defect", and the Discussion-Review rubric's own bullets raise nothing else.

## Rejected

- D11, where launch time comes from.
  The record names figures it will hold: `RunRecord` in `internal/gateslot/runlog.go` already carries `Slug`, `TierList`, `ScopeForm`, `WaitStart`, `Start`, `End` and `WallSec`, so slot wait, wall time and the pre-D3 grouping by tier list and scope form come from records that exist.
  Launch time comes only from the launch list this task adds, and D11 says so twice: "A record written before this task carries no launch list", and the auto-mode assumption that launch time and its share come from runs after the deploy.
  `goJSONCounter.handle` in `internal/gatecli/counters.go` reads only `Action`, `Test` and `Output` today, so D11 naming `Package`, `Time` and `Elapsed` as new decoded fields is the right extent of the change, and "the rendered log stays the events' output only" matches what `handle` writes.
  The `start` evidence holds against the toolchain source: `cmd/go/internal/test/test.go` creates the converter with `test2json.Timestamp`, whose `start` event is written when the package's test action begins, after its binary is built, and `Elapsed` on the package's closing event runs from that same converter's start, so the share stays at or below one.
  A cached package prints its `(cached)` output after `start` and never execs, which D11 leaves out.
- D13, a hub that keeps `covers: [0]`.
  The Verify line D13 prescribes fails "when any tier there still carries a `covers` entry", independently of the tag check, so a hub that names every `tierN` tag and keeps `covers` does not pass.
  `lyx config <module> --resolved` prints the merged document (`internal/configcli/doc.go:22`), so a `covers` key in any layer is visible to the line.
  The reasoning behind the check is right: `dropCovered` in `internal/gatecli/tiers.go` drops a covered tier by the `Covers` list alone, and `tier1`'s `-tags tier1,integration` no longer compiles `tier0`-tagged files.
- D8, `pause_after` and the entered-row mark on a skipped row.
  `routedState` in `internal/shedengine/run.go` pauses on `PauseAfter == def.Name` and `persistTransient` clears both conditions when a pause fires, so D8's "fires as for a Done" over a skipped row is the same code path with the skipped row as `def`.
  The paused row is the routed-to row, which is not entered, so a resume evaluates its gate.
  A route back into an entered row clears the mark in the routed persist, so the gate is evaluated, including through skipped rows.
  A setting changed during a pause reaches the gate, since loom steps run as child processes with a fresh `configreg.SettingLookup` per wiring and batten's `Env.Setting` builds a new lookup per call.
- D14, a second and later reconcile.
  The migration fires only while the hub file holds `tree_scope_verify`, and the pruned key is then absent, so a later reconcile changes nothing and leaves operator-added trees alone.
  The order that makes the first pass see the key holds: in `internal/configsync/configsync.go` a module's `Migrate` hook runs before `stripBase` (lines 933 and 950), and after the board's base-sparse marker exists a unit set equal to the template stays as a pin (`internal/hubreconcile/doc.go:31`), so `--set tree_scope_verify=` is still held empty when the migration reads it.
  The darn module has no `Migrate` hook today, so D14's "darn's config module's migration" is new code, which the Constraints name.
- D5 and D13, the gate-only tag guards.
  `namesGateOnlyTier` in `internal/planparser/verifymodule.go` compares tags by equality with `tmux` and `llm`, and the raw go row in `internal/shellguard/rows.go` matches `(tmux|llm)` at the end of the `-tags` value, so both refuse `tier2` and `tier3` only once extended; D5 extends both, and the acceptance criterion "A card Verify line and a raw `go test` naming `-tags tier2` or `-tags tier3` are refused" pins it.
  `tier1` stays unrefused as `integration` is today.
- D7, the guard step's non-exempt runs.
  The claim that each package of a guard group holds a guard test its filter names holds on the host: `guardNamesInFile` in `internal/impactset/guards.go` refuses a guard marker in a file the host does not compile, so no platform-constrained guard file reaches a group.
