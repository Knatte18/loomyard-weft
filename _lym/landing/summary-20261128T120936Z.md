# Name the cause whenever an orch reads as down

Issue #232 reported a running beam orch that read as down, so its members' notices went to the prime's orch.
The cause cannot be read from the code: every caller reads an orch's liveness through `orchroute.Live`.
That is the same reed `Status` read `lym orch status` makes, over the same reed geometry the orch was spawned with.
Before this change, the read returned a bare `false` for every down reading,
so no log line or status field said why an orch read as down.
This change names that cause everywhere the read happens, so the next report shows which case it was.

## What changes

**reed.**
`reedengine.StrandStatus` gains `Down`, a `DownCause` naming why a strand is not live, empty for a live one:
`pane-dead`, `pane-unlisted` (no listing carries the pane, or the strand has none),
`pane-elsewhere` (the pane sits in another session of the reed server),
or `start-shell` (the pane's start shell is in the foreground past the launch grace).
`Status` derives it from the same pane sets it already computes `Live` from.

**orchroute.**
The liveness read now produces a `Reading` with `Live` and `Down`.
On top of reed's causes it names three of its own:
`no-strand-recorded`, `no-reed-session`,
and `strand-not-in-reed-state`, an orch whose state records a strand that reed's state no longer holds.
`orchroute.Live` keeps its signature, so the notice routing, `creel post --parent`, the dobby's lines and the decision checks all use it unchanged.
It logs every down reading with its worktree and cause:
at Info for an orch that started, at Debug for one that never did.
`ReadingOf` maps an orch state and reed's strand list to a reading, and both the probe and `lym orch status` call it.

**`lym orch status`.**
The envelope gains `strand_down`, the reading's cause, empty while the strand is live.
The verb's help lists the values.

## Design decisions

- No fix to the liveness rule itself.
  One candidate was reading a strand whose pane sits in another session as live.
  Nothing shows that this caused #232, and a guessed rule risks reading a dead strand as live.
  Naming the cause makes the next occurrence decisive instead.
- An orch whose strand is missing from reed's state already read as down and passed its notices up the beam chain.
  The change keeps that routing and only names the cause.
- `no-strand-recorded` is logged at Debug: a beam whose orch never started is common, and an Info line for each probe of it would be noise.
- `lym beam status` also reads through `orchroute.Live`, so it logs a started orch's down reading at Info as well.

## To check by hand

- The #232 scenario has not been replayed in a hub with a running beam orch.
  The next time a live beam orch reads as down, the `orch: the orch reads as down` log line and `strand_down` should name the case.
- No repository-wide test run was made.

## Merge danger

**Door:** two-way
**Blast radius:** orch
