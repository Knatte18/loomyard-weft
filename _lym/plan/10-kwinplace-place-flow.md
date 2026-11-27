# Card 10 — kwinplace-place-flow

**Create:**
- `plan:internal/kwinplace#Request` -> `type Request struct`
- `plan:internal/kwinplace#Mode` -> `func Mode(perTask, machine string) string`
- `plan:internal/kwinplace#Flow` -> `type Flow struct`
- `plan:internal/kwinplace#NewFlow` -> `func NewFlow(runner *Runner, afterPlace func(ctx context.Context) error) Flow`
- `plan:internal/kwinplace#Flow.Place` -> `func (f Flow) Place(ctx context.Context, req Request) error`
- `plan:internal/kwinplace#Flow.awaitParent` -> `func (f Flow) awaitParent(ctx context.Context, placesPath, parent string, deadline time.Time) bool`
- `plan:internal/kwinplace#TestMode` -> `func TestMode(t *testing.T)`
- `plan:internal/kwinplace#TestFlow_Place` -> `func TestFlow_Place(t *testing.T)`

**Edit:**
- `cmd/lym#unboundedWaitFiles`

**Extends:**
- `internal/kwinplace/doc.go`
- `//pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/kwinplace#acquireWithin`

**Intent:** The placement child's whole flow (D7, D8, D9, D10, D12), run under one call.

`Request` carries the hub's Bolt dir, the worktree, its title and the window's pid, `Prime` (the window is the prime's own),
the parent worktree's name (the prime's worktree name for a beam or a task under the prime, empty only for the prime's own window),
the resolved mode, the two tile sizes and the hub's worktree names.
`Mode` is D7's choice: the per-task value when it is not `""`, else the machine key's.
`Flow` holds `Query`, `Apply` and `ReadTilingOutput` function fields, which `NewFlow` fills from the runner, the same module;
the `afterPlace` callback; the clock and sleeper; the path of `kwinplace.lock` beside the runner's script lock;
and the bounds: window wait 10 seconds, parent wait 180 seconds, lock wait 120 seconds.

`Place` runs, all outside `kwinplace.lock` until step 4:
1. `RecordOpen`, before any wait.
2. Read the tiling output, then the window wait: `Query` runs until a snapshot shows the window's `Window.Matches`, for at most 10 seconds,
   each run starting at least one second after the previous one ends.
   The first run passes the wait's start as `after`, and each later run the previous run's end (D10).
   The first snapshot's probe failing settles the entry unplaced and warns that placement is unsupported (D1).
   `ErrNoTilingOutput` settles it the same way, the warning naming the output.
3. Under mode `off`, `RecordReport` from the snapshot with nothing moved, and finish (D1).
4. The window needs an anchor unless it is the prime's own or its entry's home is a desktop the latest snapshot holds (D8: a homed window goes back there).
   A window that needs one runs `awaitParent` until the parent's entry is settled.
   Then it takes `kwinplace.lock` through `acquireWithin` with `lock.AcquireWriteLock` and what is left of the 120-second lock budget,
   the time spent waiting for the lock summed over every acquire of this child (D10).
5. Under the lock, a fresh `Query` passing the time the lock was taken as `after`, so no report from before the lock is reused.
   A window that needs an anchor then runs `Anchor`; a missing or unsettled parent entry releases the lock and returns to step 4's wait,
   whose 180-second bound runs from the child's first entry into it and covers every return (D8).
   A homed window whose home the fresh snapshot no longer holds needs an anchor from then on.
   Past the wait's bound the current desktop is the anchor, flagged as the fallback, so the window takes no home.
   The prime's own window anchors on the current desktop.
6. `Plan`; an empty target settles the entry with the plan's warning, releases the lock and makes no `Apply`;
   otherwise `Apply`, then `RecordReport`, then release the lock.
7. Run `afterPlace`, on every exit of steps 2 to 6, whatever ended them (D12).
A timeout, failed run, failed `Apply` or failed probe settles the entry where the last snapshot showed the window (D9), logs a warning, and still reaches step 7.
A cancelled `ctx`, the child's SIGTERM, settles the entry and returns without starting `afterPlace`;
an `afterPlace` already running gets `Place`'s `ctx`, so the cancel ends it through `desktophook.AfterPlacement` (card 3, wired on card 11).
`Place` returns the warning's error for the caller to log; it never fails the window.

`awaitParent`, in its own file `internal/kwinplace/parentwait.go`, re-reads the places file from once a second, doubling to eight seconds while the file is unchanged,
and back to one second when it changes, until the parent's entry is settled or `deadline` passes.
It is an unbounded wait for `PATTERN-daemon-wakeups`, so `parentwait.go` joins `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go`,
and the background's list gains a line for it.
The file holds no sub-second literal.

`TestMode` is `tier0`: `""` falls to the machine key's `same`; `next` wins over the machine key's `off`; `off` wins over `same`.

`TestFlow_Place` is `tier0`, rows over hand-built snapshots and reports fed through the function fields, a fake clock and a recording sleeper,
real locks and places files under `t.TempDir()`, and an `afterPlace` that counts its calls and records the context it got.
A row that needs a parent unsettled under the lock has its `Query` field, on the locked call, run `RecordOpen` for the parent with a new pid, re-opening its entry.
- a window that never appears settles its entry, warns and runs `afterPlace` once (D10);
- a lock another holder keeps past the budget settles, warns and runs `afterPlace` once;
  a child that returns to the parent wait shares one lock budget across its acquires (D10);
- the window wait's queries start at least a second after the previous one ends, the first passing the wait's start as `after` (D10);
- the locked query passes the time the lock was taken as `after`, so a shared report that ended before it is not used (D10);
- two flows started together run their apply steps one after the other (D10);
- a successful placement calls `Apply`, then `RecordReport`, then `afterPlace` once (D12);
- a failed `Apply` settles the entry, warns and runs `afterPlace` (D9, D12);
- an empty plan target makes no `Apply`, releases the lock and runs `afterPlace` (D8);
- mode `off` records the place and calls no `Apply`, then runs `afterPlace` (D1, D12);
- a failed probe, and a report of no tiling output, leave the window unmoved, settle it and run `afterPlace` (D1, D12);
- a window whose home the snapshot holds goes there without waiting for its unsettled parent (D8);
- the prime's own window waits for no parent and anchors on the current desktop (D8);
- a beam's window, whose parent is the prime's worktree, waits for the prime's entry and anchors on the desktop the prime's placement settles it on (D8);
- a parent whose placement settles its entry during the wait anchors the child on the desktop it was placed on;
  a parent unsettled under the lock sends the child back to the wait, and it anchors where the parent settles,
  including a parent under `off` and one overflowed from a full home; every return stays within one 180-second bound (D8);
- with no parent entry, past the wait the current desktop is the anchor and the entry takes no home;
  the re-reads sleep 1, 2, 4, 8 and 8 seconds, and a change to the places file sends the next sleep back to 1 (D8);
- a cancelled context before `afterPlace` runs no `afterPlace` and settles the entry;
  a cancel while `afterPlace` runs cancels the context `afterPlace` got (D12).
They add under 1 second to one `tier0` run.
`doc.go` gains a paragraph on the flow and its locks.

**ImpactSummary:** One new file joins the daemon-wakeups scan's declared list and its background; the rest is new code in the new package.
