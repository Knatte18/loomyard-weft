# Decision record — tender

## Goal

A `tender` module holds hub-wide control over what runs: pause, resume, revive after a network outage, one status view of every live session, and the stale-binary listing.
`lyx pause` and `lyx resume` stay as shortcuts.
A pause no longer consumes the deadline of a GitHub call it holds (issue #606), and the long-lived lyx processes that can restart safely pick up a replaced binary on their own.

## Scope

In:

- A new cobra module `tender` with the verbs `pause`, `resume`, `revive` and `status`.
- `lyx pause` and `lyx resume` as shortcuts in `internal/shortcutcli/shortcuts.yaml`; the top-level `pause` and `resume` root commands go.
- The pause-held deadline fix for GitHub calls, and the transient re-step of a GitHub deadline error pinned by a test (#606).
- Self-restart on a replaced binary for `lyx orch watch`, and removal of the lingering `lyx reed attach` process.
- A hub-wide listing of lyx processes running a replaced binary, inside `lyx tender status`.
- Docs, PATTERN background, test budget and sandbox coverage that the rename and the new verbs touch.

Out:

- Drain (finish every step in flight, then stop).
- A cap on concurrent runs; it stays an open point of `orch-split`, whose Go run driver is what starts runs.
- Restarting a process that is mid-work (`lyx batten run`, `lyx loom step --until-stop`, `lyx shed step --until-stop`, agent sessions): these are listed only, and their existing build-change handling is not touched.
- A restart verb.
  This narrows the board entry's "listing and restart" on purpose: every process that can restart safely restarts itself (the reed watchdog, the status panes, and the orch watcher after D8), and the rest must not restart mid-work.
  A self-restarting process that stays in the listing is a defect to report through `lyx selfreport create`.
- Changing the reed watchdog's or the status panes' binary-change handling, which `reed-window-order` built.

## Decisions

### D1 — Module layout: `internal/tendercli` over `internal/tenderengine`

`internal/pausecli` is renamed `internal/tendercli` and `internal/pauseengine` is renamed `internal/tenderengine`; `internal/pauserecord` keeps its name and role.
`tendercli.Command()` mounts one root command, `tender`, whose subcommands are `pause [<slug>] [--grace <seconds>]`, `resume [<slug>]`, `revive [<slug>]` and `status`.
`InstallNetworkGate` moves with the CLI package, and the root's pre-run calls it from there.
Rationale: `lyx --help` then lists modules only, and `pauserecord` is a leaf imported widely whose name still describes what it holds.

### D2 — Shortcuts `pause` and `resume` only

`shortcuts.yaml` gains `pause: [tender, pause]` and `resume: [tender, resume]`.
The shortcut forwards the operator's slug and flags to the end of the argv, so `lyx pause <slug> --grace 10` keeps working.
Refusal texts and stencils that name `lyx resume` or `lyx pause` keep doing so: the shortcut is the stable operator spelling.
`revive` and `status` get no shortcut.
Rationale: the board names these two; the others are rarer and stay discoverable under `lyx tender`.

### D3 — `status` replaces `lyx pause --status`

`lyx tender status` reports, read-only:

- the live pause entries, the network operations in flight and the holds, as `lyx pause --status` does today;
- every live agent session in the hub: pair, strand name, session state with its cause and since time, and whether a live pause entry covers it;
- the stale-binary listing of D7.

`pause` loses its `--status` flag, with no alias, and every place that names `lyx pause --status` names `lyx tender status` instead, `tools/sandbox/SANDBOX-CORE-SUITE.md` and `docs/overview.md` included.
`status` runs from any worktree of the hub, and any caller may run it, agents included, since it writes nothing.
Rationale: one place answers "what is running, waiting, stuck or stale"; `lyx batten overview` stays per run, `lyx reed status` per worktree.

### D4 — `pause` and `resume` behave as today

Their sequences, caller rules, inputs and envelopes are unchanged apart from the rename and D3.
Rationale: the task moves them; it does not redesign them.

### D5 — `revive [<slug>]`: one carry-on line to each session stalled on an API error

`revive` covers the hub for no slug and one pair for a slug, with the same caller rule as `pause`: the orch and an external caller may name any scope, and an agent only its own pair, where its own session is busy and only its siblings can be revived.
It enumerates the covered sessions the way `pause` does and acts on each whose session state reads idle-stalled with cause `api-error`:

- a session a live pause entry covers gets no line and is reported as paused, since `resume` gives it its line;
- the orch's session gets no line from tender: tender queues an `orchengine` notice, which the orch watcher types at the orch's next idle, and queues none while an undelivered revive notice is already queued; the report says whether a watcher is live, so a queued notice with no watcher reads as such;
- every other session gets one fixed line, declared once in `tenderengine`, through shuttle's guarded send; a send the guard refuses is reported with its reason and not retried.

A pause added while `revive` runs must not let the line reach a session it covers: the plan confirms shuttle's guarded send holds a send under a live pause entry, and otherwise `revive` re-reads coverage immediately before each send.
The report lists each session it saw, with what it did: sent, queued for the orch, paused, refused with reason, or skipped because it does not read an API-error stall.
Bound: `revive` types one fixed line, only into a session whose state reads an API-error stall at the time of the send, and only through the guarded send or the orch notice queue; it sends no key, kills nothing and changes no record.
A second `revive` finds a revived session busy or done and sends nothing, and finds the orch's revive notice still queued and queues none.
Rationale: shuttle's automatic continue covers an API error across a machine sleep only; a network outage without a sleep today needs `lyx shuttle send` per session by hand.

### D6 — A GitHub call held by a pause gets its full deadline after release (#606)

`pauserecord.Network` reports how long it held the call.
`githubclient` drops the http client's own `Timeout` and applies its 30-second client timeout itself in the transport, starting after the hold, so the request context carries only the caller's deadline.
When a call was held, the transport runs it under that deadline moved later by the held time, with the caller's values and with every cancellation of the caller's context other than its deadline still ending the call.
The extension logic lives in `githubclient`; `pauserecord` only reports the held time and keeps its import set.
Bound: the extension equals the time the gate held that one request, so an unheld request keeps exactly its deadlines, and a caller issuing several requests under one deadline gets the extension for the held request only; `landingshed` derives one deadline per call.
A pause that starts after the gate admitted a call does not hold or extend it.
`gitexec` already holds a remote command before its deadline starts, and stays as it is.
A GitHub deadline error stays classified transient (`githubclient.IsTransient`); with the hold no longer eating the budget, such an error means the call's own budget ran out after release, and the loom drive rules re-step it once before the tackler judges it, so a first occurrence never halts the run `failed`.
`loomengine.ClassifyHalt` keeps labelling a `failed` halt `producer-hard-failure` once that re-step is spent.
Any wall-clock budget the plan finds bounding a GitHub call above the transport subtracts pause coverage through `pauserecord.Excluded`.
Rationale: the hold sits inside the transport, under deadlines its callers set before the call, so the transport must own both deadlines to move them.

### D7 — Stale-binary listing

`tenderengine` lists the lyx processes of the hub whose executable was replaced: on Linux, a process whose `/proc/<pid>/exe` link names a deleted file whose base name is the lyx binary's, wherever that binary lives (the Go bin dir and `.dev-bin` alike), and whose working directory lies under the hub, a deleted working directory included.
Each listed process shows pid, start time and argv; the listing attributes no process to a pair.
On a platform with no such reading, the listing reports that it is unsupported, and the rest of `status` is unaffected.
The process reading is a seam told to the engine, implemented beside `internal/proc`.
Rationale: the operator needs to see what still runs an old binary after a deploy; the list makes a self-restart that did not happen visible.

### D8 — `lyx orch watch` re-execs itself on a replaced binary

The orch watcher calls `selfexec`'s check from its own poll and re-execs at a point where no cycle step is in flight, so the persisted cycle state carries over.
The watch lock does not survive the exec, so the new image takes it again through the watcher's existing acquire with retry.
Bound: in that gap `lyx orch start` may spawn a second watcher; both contend for the one lock and the loser exits as already running, so exactly one watcher remains, and a `refresh` or `distill` in the gap reports `watcher_live: false` and withdraws its request, as it does with no watcher.
Rationale: the watcher is a long-lived daemon whose state is on disk, so a restart in place is safe.

### D9 — `lyx reed attach` replaces itself with the tmux client on Unix

On Unix, `reed attach` execs the tmux attach argv in place of spawning it as a child, so no lyx process outlives the handover; on Windows it keeps today's child process.
The exec skips the root's post-run, so the logger is flushed and the exec logged before it happens (`PATTERN-spawn-observability`); the process's run directory is left to the next start's sweep, as for the status panes' re-exec.
Bound: nothing runs after attach in that process, so `reed attach` must be the last argv of any shortcut; the shortcut guard in `cmd/lyx` checks it, and `prime` already has it last.
Rationale: the lyx parent only waits on tmux, so removing it is simpler than restarting it, and an attach then never runs an old binary.

## Constraints

- `PATTERN-cli-cobra`: `tender` is one cobra subtree; its background's `pausecli` deviation line is replaced by `tendercli` using `tenderengine` with one root command.
  The interactive-handoff exception for `attach` already covers D9.
- `PATTERN-cliwire-sole-wiring`: `tendercli` keeps `pausecli`'s documented reason for resolving the hub through `lyxcwd` itself.
- `PATTERN-told-geometry`: `tenderengine` resolves no path and spawns nothing; the process reading, the session enumeration, the send and the notice queue are seams.
- `PATTERN-agent-name`: pair and role of a session or caller are parsed through `internal/agentname` only.
- `PATTERN-pane-text-injection`, `PATTERN-orch-pane-single-writer`: see D5.
  `creel` plans to own every delivery into a pane; revive reaches panes through the same send and notice seams `pause` and `resume` use, so creel's wiring replaces all three together.
- `PATTERN-no-denied-recovery`: every new refusal names a way forward an agent may run.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `tender` or excludes it with a reason, replacing the entries for `pause`.
- `PATTERN-daemon-wakeups`, `PATTERN-log-levels`: the watcher's binary check adds no wake of its own and logs repeated polls at Debug.
- `PATTERN-leaf-packages`: `selfexec` and `pauserecord` keep their import sets.
- Docs in the same commit: `doc.go` of both renamed packages, the module table in `docs/overview.md`, and `pattern/PATTERN-cli-cobra.md`.
- `cmd/lyx/testdata/test-budget.yaml`: the `pausecli` and `pauseengine` rows are renamed, and raised where the new verbs' tests need it.

## Auto-mode assumptions

Every decision D1–D9 and the Scope's Out list are the writer's own picks, made with no operator answer; the support log's question ledger names each.

## Open risks

- D6 depends on `net/http` merging the client's `Timeout` into the request context; dropping that timeout in favour of the transport's own is what lets the transport tell its deadline from the caller's cancellation.
  A GitHub client built elsewhere with its own `Timeout` would bypass the fix; the plan checks that `githubclient` builds every client.

## Acceptance criteria

- `lyx tender pause`, `resume`, `revive` and `status` exist; `lyx pause` and `lyx resume` run them with slug and flags forwarded; `lyx --help` lists no top-level `pause` or `resume` module.
- A GitHub call held by a pause longer than both its deadlines succeeds after resume when the server answers in time.
- An unheld GitHub call keeps its deadlines unchanged, and a call admitted before a pause starts is neither held nor extended.
- A held GitHub call whose caller's context is cancelled during or after the hold ends with the cancellation.
- A transient GitHub failure at a landing producer is re-stepped and does not halt the run `failed` on its first occurrence.
- `revive` sends its line to an idle-stalled API-error session, not to a busy, done or differently stalled one, not to a paused one, and queues a notice rather than typing for the orch; a guard refusal is reported.
- A pause added while `revive` runs keeps the line from every session it covers.
- A second `revive` before the orch's notice is delivered queues no second notice; `revive` with no orch watcher live reports the notice as queued with no watcher.
- `revive` with nothing stalled reports an empty result and succeeds; a slug with no pair is refused naming the slugs that have one; an agent naming another slug is refused.
- `status` shows pause entries, holds, sessions with their states, and the stale listing; with no pause, no sessions or no stale process each part reads empty.
- The stale listing names a process whose binary was replaced, under the Go bin dir and under `.dev-bin` alike, and omits one whose binary is current; on an unsupported platform it reports unsupported.
- The orch watcher's binary check fires at a point with no cycle step in flight and not mid-cycle, tested through `selfexec`'s exec seam, since a test binary never execs.
- A watcher re-exec racing `lyx orch start` leaves exactly one watcher.
- `reed attach` on Unix leaves no lyx process behind once tmux is attached, and a shortcut with an argv after `reed attach` fails the shortcut guard.

## Notes for the plan writer

- The hold is in `internal/githubclient/transport.go` (`authRT.RoundTrip`); `internal/gitexec/gitexec.go` `runCore` shows the hold-before-deadline shape for git.
- See `internal/shuttleengine/hold.go` for shuttle's sleep-only API-error continue.
- `selfexec` is already used by `internal/statuspane`.
- The orch watch lock and its acquire are in `internal/orchengine/watchloop.go`; `QueueNotice` is in `internal/orchengine`.
- `spawn-audit-log` plans a recorded launch path and a process sampler; D7's process reading and D9's exec are what it will later cover.
