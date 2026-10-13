# tender: hub-wide pause, resume, revive and stale-binary restart

`tender` · Task · enhancement, bug, shuttle, reed, batten, shed, orch, loom · running · Discussion-Write

## Idea (operator, 2026-10-13)

`lyx pause` and `lyx resume` are two top-level verbs from one package (`internal/pausecli`), so `lyx --help` lists them as if each were a module.
They should live in a module of their own, named `tender` (a loom tender keeps the looms running: stops them, restarts them, watches over them), with `lyx pause` and `lyx resume` kept as shortcuts (one line each in `internal/shortcutcli/shortcuts.yaml`).

Names ruled out (operator, 2026-10-13): `hub` (the whole LYXHUB folder), `treadle` (taken by `internal/treadleengine`), `weaver` (planned as a sister module of Webster), `brake` and `hold` (fit pausing only, not restarting sessions).

## What else could live in `tender`

Hub-wide control over everything that runs, much of it done by the orch by hand today:

- `pause` and `resume`, for the hub or one pair, as today; a pause must not eat a deadline (see the folded-in section on issue #606).
- Revive after a network outage: find every session with `cause: api-error` and `state: idle-stalled` (today `lyx shuttle state` in each pair) and give each one line to carry on (today `lyx shuttle send` per session).
- Drain: let every step in flight finish, then stop, unlike pause, which interrupts at once; for a deploy or a hibernation.
- Stale binaries: list long-lived processes running a deleted binary and restart them (see the folded-in section on `stale-binary-daemons`).
- Status: one view of every live session in the hub, who is working, waiting or stuck.
- A cap on concurrent runs (an open point in `orch-split`).

## Scope to discuss

- Which of these belong in `tender`, and which stay with their modules (`shuttle`, `reed`, `batten`).
- Verb names under `tender`, and which get shortcuts.

## Folded in: pause-held-deadline (issue #606)

Imported from [issue #606](https://github.com/Knatte18/loomyard/issues/606).
Bug: a deadline keeps running while a pause holds a call, so PR-Align failed after `lyx resume`.

### What happened

A run halted `failed` at `PR-Align` with anomaly `producer-hard-failure`:

```
landingshed: PR-Align: query existing pull request: context deadline exceeded
```

The trace shows the sequence:

- `alignment: spawn` for `point=pr-gate` with a 20-minute `deadline`.
- `pausecli: a network call is held by a pause` — `lyx: held by a hub-wide pause since ...; it continues after "lyx resume"`.
- About 70 minutes later the pause ended, the session went `busy` again, and the first `landingshed: github call failed` for `action="query existing pull request"` reported `cause="context deadline exceeded"`.

### Why it is abnormal

The deadline kept running while the call was deliberately held by `lyx pause`.
The operator's pause, not a GitHub or network fault, consumed the budget, so the first call after `lyx resume` failed at once and the whole run halted as a hard failure.
The alignment session itself finished `done` (`loop=done`), yet the run was still marked `failed`.

### Scope

- Do not count pause-held time against the deadline: extend the context deadline by the pause duration, or start the deadline only after the pause releases the call.
- Treat a deadline error on a call that was held by a pause as retryable, not as `producer-hard-failure`.

Provenance: task `shuttle-gate-early-done`, note `loom-halt.md`.

## Folded in: stale-binary-daemons

Bug: long-lived lyx processes keep running the binary they started with after a deploy replaces `lyx`, and nothing restarts them.
They should pick up the new binary on their own.

This reuses the shared re-exec helper that `reed-window-order` builds (`internal/selfexec`, per its Discussion), hence the dependency on `reed-window-order`.
The status panes and the reed watchdog moved to `reed-window-order` on 2026-10-13; what remains here is the rest of the long-lived verbs and the hub-wide listing and restart.

### Seen (2026-10-13)

- After the watchdog kill, other lyx processes still ran a deleted binary (`/proc/<pid>/exe` marked deleted): `lyx orch watch` (since 2026-10-10) and an operator's `lyx reed attach`.

### Scope to discuss

- Which processes restart themselves: `lyx orch watch`, an operator's `lyx reed attach`, and any other long-lived verb.
- How: each process notices a changed binary and re-execs itself with the same arguments at a safe point (using the shared helper from `reed-window-order`); or the deploy (`update-plugins.sh` and batten's Land deploy) signals or restarts every running process.
- Processes that must not restart mid-work (an agent session, a step in flight) are only reported, for instance in `lyx reed status`.
