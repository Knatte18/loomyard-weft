# A pause holds operator verbs, not only runs: the network gate covers every git and GitHub call

`pause-holds-operator-verbs` · Task · bug, tender, git, batten, selfreport

Imported from [issue #129](https://github.com/Knatte18/loomyard/issues/129).

## Seen

A `lym tender pause` holds more than runs.
While a pause stands, operator verbs that are not runs wait on it.
#124 reports this for `lym selfreport create`; it is wider than that.

Seen on one machine while the hub was paused:

- `lym selfreport create` was held with `lym: held by a hub-wide pause since <time>; it continues after "lym tender resume"`.
  `lym cleanup` hung the same way, but its held line was not seen.
- In a pair's worktree that a run-scoped pause covered, `lym s` printed `outcome: owned_here` and then:

```
lym: held by the pause of <slug> since <time>; it continues after "lym tender resume" or "lym tender resume <slug>"
```

  `lym s` is `lym adopt --here --no-orch`, then `lym beam start`, then `lym reed attach`.
  In that worktree the first step is only an owner check, not an adopt, yet the start that follows is held.
- `lym board get <handle>` and `lym machine name <name>` hung from the paused hub's worktree; `lym board find` and `lym board list` answered.
  Whether the pause held those two was not confirmed: no held line was seen.

A held call writes its line once, to stderr, and then sleeps.
A caller that discards stderr or has a timeout sees only a hang or a kill; one without a timeout waits as long as the pause stands.

## Why it makes no sense

A pause stops runs: agent turns and the network work they do.
It is not meant to stop the operator's own verbs, and a pause is often set because something is wrong, which is when `selfreport`, `board get` and `machine name` are needed most.

## Where it comes from

`internal/tendercli/gate.go` installs one process-wide gate (`pauserecord.InstallGate`) in every lym process that runs in a hub.
Every git call goes through `pauserecord.Network` in `internal/gitexec/gitexec.go`, and every GitHub call through the same function in `internal/githubclient/transport.go`.
Both wait in `pauserecord.WaitUncovered` while any live pause entry covers the process.
So any verb that reaches origin, even to fetch the board before a read, waits on a pause that has nothing to do with it.

The gate has an exemption, `SetGateExemptAll`, "for the one process an operator's explicit verb runs in".
Only `lym adopt`, the fabric teardown and `lym batten cancel` call it.
Every other operator verb is held.

A run-scoped pause is held at a second place: `internal/battenshed/innerrun_pause.go` holds the spawn of a covered run, and that wait is what `lym s` reached above.

## Idea

Invert the default: the gate holds only the processes a pause is meant to stop, which are the run's own processes (a strand's agents, the loop and its children), and an operator verb run from a terminal passes.

- Install the gate with `ExemptAll` for a process that is not a run process, decided where `InstallNetworkGate` is called, so no verb has to opt out one by one.
- Keep holding detached children of a run, which `DetachedChild` already marks.
- Read first which verbs outside a run call `InstallNetworkGate`, and which of them a pause is meant to hold, such as a push child of a run.
- Add a test that a paused hub does not hold `board get`, `board list`, `machine name`, `selfreport create` and `lym s`.
- Make a hold visible: a verb that does wait prints the held line at once instead of staying silent.


## From issue #124

Imported from [issue #124](https://github.com/Knatte18/loomyard/issues/124).

## Seen

With the hub under a hub-wide pause (`lym tender pause`), `lym selfreport create` does not file the issue.
It waits and is held until the pause is lifted, with this message:

```
lym: held by a hub-wide pause since <time>; it continues after "lym tender resume"
```

Nothing is filed meanwhile, and a caller without a timeout hangs.
Two earlier calls on the same hub hung this way, for as long as the pause stood.

## Why it makes no sense

A pause holds agent and network work so runs stop.
`selfreport` is the way to report that lym is wrong, and a pause is often set because something is wrong, so the report is blocked when it is most needed.
Filing one issue has nothing to do with the runs the pause protects.

## Idea

Exempt `lym selfreport` from the hub-wide pause, so it files at once while the hub is paused.
Read first where the pause holds the call, and whether other operator verbs that only report, such as reads and `lym report`, are held the same way.
