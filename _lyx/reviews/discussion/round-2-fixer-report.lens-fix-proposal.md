```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer proposals the fixer applied as written.

**MEDIUM**: one of the two ways forward that the Finalize halt now names is refused by the run lock.
- **Changed text:** D9's Finalize bullet: "halts naming, for each member, only `lyx batten cancel <member> --remove` or moving its `project` off the beam with `lyx board upsert`, then `lyx loom resume`". The acceptance criterion "an open member Finalize finds halts the pawl run naming its cancel or its move off the beam" repeats it.
- **The case:** a member that has a pair usually has a run in flight, so its entry's status is a run status. `checkRunLocks` refuses any `scopeChanged` edit of such an entry, and `scopeChanged` compares `before.Project != now.Project` (`internal/boardengine/runlock.go`). The `lyx board upsert` way forward therefore fails for exactly the members most likely to trip the backstop.
- **What the proposal lost:** today's `beamLandRefusal` (`internal/battencli/beamland.go`) splits members by `HasPair`. A member with a pair is told to cancel; a member with no pair here is told to move its `project` off the beam. The proposal offers both options to every member.
- **Needed:** per member, `lyx batten cancel <member> --remove` when it has a pair or a run in flight, and the `project` move only for a member with no run status.

**LOW**: the driver-dead notice wording is wrong, and the new criterion may be unreachable.
- **Changed text:** D9's InnerRun bullet: "sends the driver-dead notice at Beam-Await as at any row (`notice.go`), naming `lyx loom start` in the beam worktree". Also the new criterion "A Beam-Await driver killed more than `respawnCap` times with no status write between sends the driver-dead notice naming `lyx loom start`".
- **Wording:** `loomStartRemedy` (`internal/battenshed/notice.go:393`) reads "run \"lyx loom start\" in the task worktree". Saying the notice names "the beam worktree" claims text that the code does not send.
- **Reachability:** the respawn counter resets whenever the child's status-file modification time moves (`innerrun.go:430`, `443`). If a re-spawned driver writes its status file when it re-enters Beam-Await, the cap never trips. In that case the criterion tests a case that cannot arise. The record does not say whether re-entry writes that file.

No findings on the proposals for F1, F2, F3, F4, F6, F7, F8, F9, F10, F12 and F13. For each I built a case and it held:
- F3: a failed Parent-Align write now stops nothing, so a retry is clean.
- F2: a crash between the request's removal and the marker's removal is cleared by the interrupted-landing branch, whose notice names `lyx beam start`.
- F6: its way forward `lyx batten cancel <slug>` chains through D11's member refusal and still leads somewhere.

Tool calls:
- Read `innerrun.go:340-360` and grepped the respawn reset and `loomStartRemedy`: my context lacked the counter-reset rule and the notice text.
- Grepped for `lyx loom resume` as a way forward: I needed to confirm the verb exists for a halted run, which it does.
