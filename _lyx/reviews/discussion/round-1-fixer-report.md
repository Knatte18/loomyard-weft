# Fixer report — discussion, round 1

Every finding's premise held against the code (`awaitIdleSession` in `internal/shuttleengine/send.go`, `Runner.Send` in `internal/shuttleengine/run.go`, the prime exclusion in `internal/pausecli/deps.go`, the in-memory pause fields of `Watcher` in `internal/orchengine/watcher.go`, `WaitUncovered(context.Background(), ...)` in `internal/pauserecord/hold.go`, the single `listCtx` in `internal/fabriccli/openprheads.go`).
Every finding is fixed as proposed.

## Changes

- F1: D5's guarded-send bullet now refuses, with no idle wait, unless the session reads idle-stalled with cause `api-error` at the send's own ready reading, and reports any other reading as skipped; the orch bullet has the watcher drop a revive notice at delivery when the orch no longer reads that stall, and the report calls the notice conditional; the Bound names both readings.
  Two criteria added: a session turning busy between read and send gets no line; a queued orch notice whose orch recovered is dropped.
- F2: the "pause added while revive runs" sentence replaced by the coverage re-read before each send, with the residual window and its bound by the pause's grace-then-interrupt; criterion replaced by "a pause entry live at revive's coverage read keeps the line from that session".
- F3: D5 enumerates the task pairs' sessions plus, for the hub scope, the orch's session from the prime's orch state, and revives no other prime session; D3 now lists the prime's sessions, the orch's and any other live one; criterion added that a hub-scope revive reads the orch's state and no other prime session.
- F4: D6's `pauserecord.Excluded` sentence replaced by the per-request deadline rule for several-request callers; Bound kept; criterion added for a several-request caller whose second request is held.
- F5: D6 states the hold stays blind to the caller's context, ending a cancelled call at release, with `pauserecord.Network` keeping its context-free signature; criterion reworded.
- F6: D8's re-exec point also requires no unhandled live hub pause entry; Bound names a hub `pause` or `revive` in the gap and the new image's first-tick handling; criterion added.
- F7: D7's matching rule names the running lyx's own executable base name, strips `(deleted)` from both links, and uses path-segment containment.
- F8: D3's caller sentence states what `status` exposes to an agent and that it reads no transcript or pane content.
- F9: the status criterion names the per-session pause-coverage flag; criterion added for `pause --status` refused and no doc or stencil naming it.
- F10: the restart-verb Out bullet cut to one line; the "narrows the board entry" sentence moved to the support log's restart entry; the selfreport sentence dropped.
- F11: the Open risks bullet deleted, the section reads None, and a `PATTERN-github-auth` constraint added.
- F12: D5's orch bullet states that two concurrent revives may each queue a notice.

## Disputed

None.

## Deferred

None.
