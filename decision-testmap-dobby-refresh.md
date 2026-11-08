## Goal

The prime parent's dobby starts lym gate testmap after every test_map_every landings

Takes: testmap-verb's verb
Delivers: per the operator's decision after the Motif (2026-11-08, in the Sketch's board body), which replaces D6's trigger and supersedes the trigger clauses of the first carried criterion: a dobby tick step on the prime dobby that counts landings as the board tidy does and, after every test_map_every landings, starts `lym gate testmap --max <test_map_batch>` on the packages stale then, leaving what the cap left unbuilt for the next refresh; no prime-HEAD record and no exit-record trigger; started through a spawn dep registered as runproc.RoleTestMap under runproc.HubOwner, none while a live test-map record exists, a restarted dobby included; the child ended on stop through runproc.EndRole; dobby config keys test_map_every beside board_tidy_every, where 0 never refreshes, and test_map_batch; no test_map on/off key
Exports: runproc.RoleTestMap

## Scope

The part changes these packages:

- internal/dobbyengine (about 200 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/runproc (about 10 changed lines)

## Decisions

### D6 — The prime parent's dobby refreshes maps when the prime branch moves

The dobby whose worktree carries the hub's prime parent branch keeps its own record of the prime HEAD its last refresh started from.
On a tick where the prime HEAD differs from that record, or the last refresh reported stale packages the `--max` cap left unbuilt, and no refresh is in flight, it starts `lym gate testmap --max <test_map_batch>`.
The dobby learns of leftovers from D5's exit record alone:
once the child it waited on exits, it reads the record and takes the count only when the record names that child's process ID.
A refresh that was killed, or whose record is missing, unreadable or names another process, counts as reporting none left,
so only a prime HEAD change starts the next one.
Packages a refresh skipped never start another.
`test_map_batch` is a dobby config key with default 5.
A fresh dobby has no record, so its first tick starts a refresh.
The process is started with the binary `selfexec.LymBinary` names, logged, and waited on by the dobby that started it.
It is registered under `runproc.HubOwner` with a new role, `runproc.RoleTestMap` (new), spelled `test-map`.
A refresh is in flight while a live `test-map` record exists, so a restarted dobby reads the record and starts no second refresh.
The prime dobby's stop also calls `runproc.EndRole` for `runproc.HubOwner` with `runproc.RoleTestMap` (new), which ends the child and the processes below it.
A dobby that dies without a stop leaves the child running.
Once the child holds its gate slot it ends on its own within D5's time bound,
and its gate slot and package locks, OS locks, are released when it exits.
A dobby config key, `test_map` in `internal/dobbyengine/template.yaml`, switches the step on or off, default on.
The trigger is independent of the board-tidy landing count.

Rationale: a prime HEAD change is what makes maps stale, whatever landed it.
Bound: a package skipped as locked or changed under a landing waits for the next prime HEAD change.
A modified or untracked Go file in the prime keeps its package skipped until it is committed or removed, and the skip list names the file.
At most one refresh runs per hub, and one a dead dobby left behind lasts, once it holds its gate slot, no longer than D5's time bound.
Only the machine running the prime parent's dobby gets maps; a gate on another machine reports "no map" for each package (D7).

## Constraints

- Per-test coverage needs each test in its own process run: `runtime/coverage` counters are process-wide, so tests under `t.Parallel` mix their counts.
  The map cannot come from a normal tier run.
- No test of this feature launches a test binary or `go test`: the runner's tests use a stand-in `go` and fake profiles, as `cmd/testtiming`'s prebuild test does.
- `PATTERN-gate-slot`: the verb and the step run tests only while holding gate slots.
- `PATTERN-fabric-boundary`: the verb's tracked-file and status reads (D5) and the step's merge-base reads (D7 step 1) go through `internal/fabricengine`.
- `PATTERN-config-layers`: every new key has a default and loads through `configengine`; no meaning of an existing key changes, so no reinterpretation ships.
- `PATTERN-cli-cobra`: `lym gate testmap` joins the existing `lym gate` cobra subtree.
- `PATTERN-refusal-way-forward`: the verb's refusal outside the prime worktree and the unknown report name refusal each name their way forward, and a test reaches each.
- `PATTERN-code-conventions`: the step's keys come from `gate.yaml`, never a hard-coded key list.
- The prime parent's worktree is updated in place when a run lands, so its HEAD tracks the prime branch.
  `internal/fabricengine/merge.go`'s merge runs `MergeStart` in the parent's warp checkout under the weft write lock.

## Auto-mode assumptions

- The full design is built, not only the tool change: the goal asks that redundant tests stop piling up, which needs the check on new tests.

## Open risks

## Acceptance criteria

- D6: the dobby starts a refresh when the prime HEAD differs from its record, when D5's exit record names the refresh it waited on and a count of stale packages the cap left unbuilt above zero, or on its first tick.
  It starts none when the prime HEAD is unchanged and the last refresh only skipped packages, none while a live `test-map` record exists, a restarted dobby included, and none when `test_map` is off.
  At an unchanged prime HEAD it starts none for a refresh that was killed or whose exit record is missing or names another process.
- D6: a dobby stop ends a running refresh through `runproc.EndRole` with `runproc.RoleTestMap` (new).
