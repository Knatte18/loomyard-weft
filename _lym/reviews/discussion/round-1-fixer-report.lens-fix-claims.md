```yaml
lens: fix-claims
outcome: reported
```

Task: check that every bound, guarantee and code fact the fix added or rewrote holds against the code at HEAD.

**MEDIUM: D6's new Bound line overstates who can change presence**
- Changed text: D6 Bound, "presence is changed only by the operator, an orch, a process outside every pane, and the relay on the configured operator's Slack text; no other agent strand can move the operator's messages and reminders between the terminal and Slack."
- Problem: the refusal reads pane ancestry, so it only stops a process that is still in a non-orch pane's tree when it runs. An agent strand can leave its pane tree with the double fork D3 itself uses, for example `setsid -f lym slack back`. That process sits outside every pane, so the verb takes effect. The second clause is therefore false.
- Evidence: `internal/creelengine/identity.go` `paneOfPeer` walks parent pids only, and D3 bullet 2 of the fixed record relies on reparenting to leave a pane tree. D7's Bound already admits that "any process of the operator's user outside a pane can post as `SLACK`", so the new D6 line contradicts its sibling.
- Fix: limit the claim to "no process inside a non-orch strand's pane". Name the escape as a bound, as D7 does.

**LOW: D3's new bound rests on an unstated premise about where the relay is reparented**
- Changed text: D3 bullet 2 Bound, "the relay is never in a pane's process tree at its first read, whoever starts it".
- Problem: an orphan is reparented to its nearest ancestor that set `PR_SET_CHILD_SUBREAPER`, and only to init when there is none. The bound holds only if no process in a pane's tree is a child subreaper. The record does not state that.
- Evidence: `grep -rni subreaper` over the repo finds nothing, so lym sets none itself. The engines running in panes are not checked.

Claims checked that hold:
- The anchor change's `EndDobbies` ends every `runproc.HubOwner` process but a push or a sync. `internal/fabriccli/anchordeps.go:248` calls `runproc.EndSparing` with the owner `dobbyOwner(worktree)`, which is `runproc.HubOwner` for the prime (`anchordeps.go:85-88`).
- `retain` carries unread messages and a program's read forward. `journal.go` `retain` writes the records from `pool.carryRecords`, and `fold.go:537-543` covers the readers.
- A hold renews when the same owner holds again, so it lasts up to an hour past its last renewal (`creelengine/doc.go` Holds).
- `lym creel log --to` exists (`internal/creelcli/cli.go:764`).
- Dobby's state record is tolerant JSON under `.lym/beam/` (`dobbyengine/doc.go:9`).
- `lym slack stop` setting `terminal` first means no dobby tick restarts the relay, which matches D3 bullet 3's condition "while presence is `slack`".

Further tool calls:
1. Bash grep of `EndDobbies`, `retain`, Holds and subreaper. My context lacked the bodies of the `EndDobbies` implementation and `retain`, the hold renewal rule, and any subreaper use.
2. Bash grep of `anchordeps.go` `EndDobbies` and `carryRecords`. I needed whether `EndDobbies` covers `HubOwner` and whether a program's read is carried.
3. Bash grep for the `dobbyOwner` definition and `HubOwner`. I needed the owner the prime's dobby registers under.
4. Bash grep for the `dobbyOwner` use sites. The definition's location was still unknown.
5. Read of `anchordeps.go:80-95`. I needed `dobbyOwner`'s body to confirm the prime maps to `HubOwner`.
