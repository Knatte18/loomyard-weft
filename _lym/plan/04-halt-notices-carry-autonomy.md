# Card 4 — halt-notices-carry-autonomy

**Create:**
- `internal/orchcli#NotifyParentHalt`
- `internal/orchcli#NotifyRunFromHalt`
- `internal/orchcli#withRunAutonomy`
- `internal/orchcli#TestNotifyHalt_CarriesTheRunsAutonomy`

**Edit:**
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#newPrePushCheck`
- `internal/battencli#notifyRunParent`

**Uses:**
- `internal/hubgeom#Autonomy`
- `internal/orchcli#NotifyParent`
- `internal/orchcli#NotifyRunFrom`
- `internal/orchcli#TestNotifyPrime_QueuesInThePrimeOnly`
- `internal/orchcli/doc.go`

**Intent:** D2: every halt notice lym posts for a run carries `autonomy: <true|false>`, the run's value read when the notice is built,
so the orch the notice reaches handles it under the run's value, never its own worktree's.

The stamp and the post are one call, so a halt site cannot post the line without the stamp:

- `withRunAutonomy` returns `line + "; autonomy: true"` or `"; autonomy: false"`, the value `hubgeom.Autonomy(location, slug)` answers at the call.
- `NotifyParentHalt` posts `withRunAutonomy(pair, pair.WorktreeName, line)` through `NotifyParent`'s routing and tags.
- `NotifyRunFromHalt` posts `withRunAutonomy(from, slug, line)` through `NotifyRunFrom`'s routing and tags; it serves a run whose pair is absent.
- Both sit in `notify.go` beside the `Notify` functions, and their comments say they serve a run's halt notices.

The halt notices of a run swap their post for the halt form:

- `goDriverDepsFor`'s `HandBack.Notify`, the driver's hand-back of every Stuck, relay and question, which carries the landing, Finalize, drift and pawl halts:
  `orchcli.NotifyParentHalt(location, loomProgram, line)`.
- `notifyRunParent`, batten's row-stop and hand-back notices: `NotifyParentHalt` with the pair present, `NotifyRunFromHalt` with it absent.
- `newPrePushCheck`'s `notify`, the "stopped before its push" stop: `NotifyParentHalt`.

Notices that are no halt keep their text and their post: dobby's run notices, hold notices, landing notices and `[beam drained]`.

`TestNotifyHalt_CarriesTheRunsAutonomy` is tier1, in a file of package `orchcli`, in the shape of `TestNotifyPrime_QueuesInThePrimeOnly`:
a `hubforge.CopyHub` hub with a beam pair, a task pair added from it, and the prime's orch strand recorded, so the notice queues in the prime.

- With the task's own file setting `true` under the beam's `false`, `NotifyParentHalt` for the task queues one mail whose body is the line followed by `; autonomy: true` (D2, D11's task-own value).
- The task's file then rewritten by hand to `autonomy: false`, with no `lym config beam --set`, makes the next `NotifyParentHalt` queue a body ending `; autonomy: false`
  (D9's hand-edit criterion, D2's "read when the notice is built").
- `NotifyRunFromHalt` from the beam with the task's slug queues a body ending with the task's value, `; autonomy: false`.

**ImpactSummary:** Three post sites swap `NotifyParent` or `NotifyRunFrom` for its halt form with the same routing, tags and signer; only the posted body gains one clause, and no existing test asserts these sites' bodies.
