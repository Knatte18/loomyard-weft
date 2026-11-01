```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** Cases run for each function or branch the diff changes:

- `dobbyStartError`:
  - Started and Live return nil. Correct.
  - Refused and Failed return an error naming the worktree and `DobbyDetail`. Correct.
  - An empty outcome with warnings returns an error carrying the joined warnings. Correct.
  - An empty outcome with no warnings returns an error ending in `: ` with no detail. Cosmetic only.
- `BeamUpWorktree`, prime case: `worktree == prime.WorktreeName`, so `BeamUp(prime, {})` runs with `Prepare` false, `Here` read from the machine name and `Index` defaulted. Correct.
- `BeamUpWorktree`, beam case: `ResolveWorktree(WorktreePath(prime, slug))`. A missing worktree returns a wrapped error. Correct.
- `BeamUpWorktree`, dobby already live: `EnsureLive` reports Live, `dobbyStartError` returns nil, and `BeamUp` is idempotent for a live orch (action `already-running`). Correct.
- `BeamUpWorktree`, `EnsureLive` returning an error with no entries: the outcome is empty, `startDobbyStep` adds a warning, and `dobbyStartError` joins the warnings. Correct.
- `BeamUpWorktree`, dobby refused (other machine owns the worktree, or landing held): `BeamUp` still goes on to start the orch, and then the error is returned. See F1.
- `ensureParentDobby`:
  - The owner and landing-hold refusals still run before `start(parent)`. Correct.
  - The `startWay` text and the error wording name `lym beam start`. Correct.
  - `c.location` is passed as the prime, as the seam comment says. Correct.
  - A nil `pairStart` defaults to `BeamUpWorktree` over the receiver's prime. Correct.
  - A failed start is a warning in `startFirstPass`, and the started pass is kept. Correct.
- `pawlDeps.StartDobby` calls `BeamUpWorktree(hub, slug)`:
  - A beam this machine does not own gives a refused outcome, which becomes an error, as `startBeamDobbyAlone` did.
  - A failed outcome is now also an error. The old code returned errors only for refused entries and swallowed failed ones, so this is stricter and arguably more correct.
  - The `EnsureLive` error return is propagated, as before. Correct.
- `moveAnchor` success path: `startDobbies` now runs after `releaseParentLock()` and `change.Close()`. Correct.
- `moveAnchor` `fail` path (`internal/fabriccli/anchor.go:371-390`): the parent lock is released and the change closed before `startDobbies`, so the child's origin reconcile cannot meet the lock. Correct.
- `hubAnchorDeps` `EnsureDobbies`: `prime()` is called inside the closure, at start time. Correct. This needs the prime location after a prime anchor move, and the `AnchorPath` it feeds to `cmd.Dir` is built from `AnchorRel` (`internal/lymcwd/lymcwd.go:59`).
- `beamStart`:
  - The prime maps to the label `root`; a beam maps to its slug. Correct.
  - A `cmd.Output()` failure puts the JSON error envelope (stdout, via `finishVerb`) into the message, with `exit status N` as `%w`. Correct.
  - Under test it returns an error and starts nothing. Correct.
  - Failures are joined, so one failed worktree does not stop the others. Correct.
  - The join drops the per-worktree label when `Error()` is read, but each message names its label. Correct.

### Findings

**F1: MEDIUM, design.** `internal/orchcli/start.go:491-495`, with `internal/orchcli/start.go:496-510`.
- When the dobby step is refused or failed, `BeamUp` does not stop. A dobby that does not start only warns, and the orch and watcher are brought up regardless.
- `BeamUpWorktree` then returns an error, so the caller sees a failure while an orch is left running with no dobby, the inverse of the pair this change enforces.
- This hits a held landing, an owner on another machine, and a dobby spawn failure.
- Evidence: `BeamUp` continues after `startDobbyStep` (`internal/orchcli/start.go:471`, `498-510`), and `dobbyStartError` is evaluated only after `BeamUp` returns (`internal/orchcli/start.go:491-495`).

**F2: LOW, design.** `internal/orchcli/start.go:479-495`, with `internal/orchcli/start.go:215-217`.
- `BeamUp` starts the dobby before it touches the orch.
- If the orch step then fails, for example creel refusing the ORCHWATCH claim from a non-orch strand's pane (`withdrawWayForward`), `BeamUpWorktree` returns the error.
- A lone dobby is then already live. A caller running from a strand pane, such as batten's first pass or the anchor move child, therefore still ends with a dobby alone, only with a warning or `dobby_start_error`.
- This is inherent in the dobby-first order. It is not shown to be reachable, only possible from the withdraw text.

**Other:** I found no other defects in the changed branches.

### Tool calls
1. `grep` of `fail := func` in `internal/fabriccli/anchor.go`, to see whether the failure path releases the parent lock before `startDobbies`. My context held only the success path.
2. `grep` of `AnchorPath` in `internal/lymcwd`, and of `primeLocation` in `internal/fabriccli/anchordeps.go`, to check that the child's `cmd.Dir` is not stale after a prime anchor move. My context lacked the definition.
