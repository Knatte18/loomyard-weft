# Card 9 — goto-ends-fixers

**Create:**
- `plan:internal/loomcli#endGateFixers` -> `func endGateFixers(runProcDir string, slug func() (string, error), end func(dir, owner, role string, callerPID int) ([]runproc.Ended, error)) error`
- `plan:internal/loomcli#loomCLI.loomPreGoto` -> `func (c *loomCLI) loomPreGoto(ctx context.Context, target string) error`
- `plan:internal/loomcli#loomCLI.endRunFixers` -> `func (c *loomCLI) endRunFixers() error`
- `plan:internal/loomcli#TestEndGateFixers` -> `func TestEndGateFixers(t *testing.T)`
- `plan:internal/loomcli#TestSpecFor_PreGotoEndsFixers` -> `func TestSpecFor_PreGotoEndsFixers(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#loomCLI.loomConsumeAttempt`

**Uses:**
- `plan:internal/loomcli#pairSlugOf`
- `internal/loomcli#loomCLI.driverGoto`
- `internal/loomcli#TestSpecFor_WaitingHookNilWithoutLocation`
- `internal/loomcli/doc.go`

**Intent:**
A move of the run onto another row leaves no fixer running (D8): before it moves the run, it ends every role of `gatefixer.Roles()` registered under the pair through `runproc.EndRole`, so no fixer of the visit it leaves, a card fixer or webster's `verify-fixer` included, keeps editing the worktree under the next row.
Two paths move a run: `shedverbs.RunGoto`, behind the cobra `goto` verb and `driverGoto`, and `shedengine.Relaunch`, behind the tackler's and the arbiter's consume of a stop; both end the fixers first.

- `endGateFixers(runProcDir, slug, end)` reads the pair slug through `slug`, then calls `end(runProcDir, owner, role, os.Getpid())` for each role of `gatefixer.Roles()` in order; an empty slug ends nothing.
  A slug read error is refused with the way forward `transient, re-run the goto`; an `end` failure is returned wrapping the role and the error, with the way forward to re-run the goto once the named process is gone.
  Either refusal leaves the run unmoved, since `shedverbs.RunGoto` returns a `PreGoto` error before `shedengine.Goto`, and the consume paths below return before they relaunch.
- `loomCLI.endRunFixers` calls `endGateFixers` with `fabricengine.RunProcDir(c.location.HubPath)`, `pairSlugOf(c.location)` and `runproc.EndRole`; a receiver with no location ends nothing.
- `loomCLI.loomPreGoto` calls `endRunFixers`; `loomCLI.specFor` sets `Hooks.PreGoto` to it.
- `loomCLI.goDriverDepsFor`'s `ConsumeTackler` and `loomCLI.loomConsumeAttempt` call `endRunFixers` before they consume the attempt, so a relaunch onto another row ends the fixers first too; a refusal there is the consume's error, which the driver reports with the stop as it reports a failed consume today.

Bound: the hook ends only processes registered under the pair with a fixer role; the writer's own session and the gate runner have other roles and are untouched.

Tests, tier0:
- `TestEndGateFixers` over a recording `end` seam and a scripted slug: both fixer roles are ended under the owner with the caller's pid; an empty slug calls nothing; a slug read error is refused naming the transient re-run and calls nothing; an error from the first role stops the loop and is returned naming the role and the way forward.
- `TestSpecFor_PreGotoEndsFixers`: `specFor("goto")` carries a non-nil `Hooks.PreGoto`, and on a receiver with no location it returns nil and ends nothing, as `TestSpecFor_WaitingHookNilWithoutLocation` builds its receiver.

**ImpactSummary:** one hook on loom's verb spec and one call before each of loom's two relaunch consumes; batten's runs and every other verb are unchanged.
