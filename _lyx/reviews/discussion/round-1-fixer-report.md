# Fixer report — discussion round 1

Each finding's premise was checked against the code at HEAD, and every one held.

## Changes

- F1 (BLOCKING): D9 is now "Reed's session end stays as it is".
  It records that `Engine.Down` and `EndSessionByName` (via `reapSessionVia`) already reap each live pane's descendant closure for every session, the prime's included (`reapPaneChildren`, `lifecycle.go`, `overlay.go`).
  It also records that an unregistered lyx process such as an agent's `lyx gate test` stays killed, and that #796's shells were not at a session end, which D11 covers.
  Removed the matching Scope line, the D9 acceptance row, the D9 Open risk and D9's part of the shuttle-stop constraint.
  In the support log, rewrote the Turn 3 line and Q10, and added the dropped reap to Rejected alternatives.
  Added a Constraints fact with its source.
- F2 (BLOCKING): fixed as proposed.
  D5 adds a waiting marker (pid, start time, slug) that `pauserecord.Network` writes before `WaitUncovered` and removes before writing its in-flight marker; `ListNetOps` never lists it.
  D5 reads that marker for the run-lock holder.
  Added a Constraints fact and an acceptance row.
- F3 (BLOCKING): fixed as proposed.
  D4 states the exact session-end order, both skipped and not skipped.
  D5 puts its bounded run-lock wait, naming its holder, before `End` and before the loop end.
  D3's step-child line now says the step dies at `End`.
  Updated the PATTERN-pair-teardown text and added a Constraints fact.
- F4 (BLOCKING): fixed as proposed.
  In D3, `End` reads the descendants of leaders and non-leaders before the first signal, and signals a descendant that leads its own group with that group.
  The Rationale states the `gitexec`/`ConfigureGroupKill` fact and what happens to a push relative to `remove --remote`.
  The SIGTERM `index.lock` line is narrowed to git processes `End` signals.
  Added a Constraints fact and an acceptance row.
- F5 (BLOCKING): fixed as proposed, without a reconcile.
  D4 lists `reed down`'s local readings, defines "not running" (no driver lock, loop lock free, batten not running) and says it has no `stopped` reading and makes no network call.
  Updated the acceptance row.
- F6 (MEDIUM): fixed as proposed.
  D7 covers every owned-elsewhere pause site, batten's owner probe included.
  Batten keeps running, its removal is named, and the Bound and acceptance row are widened.
- F7 (MEDIUM): fixed as proposed.
  D4 defines batten as running only while a process registered under the slug with the batten role is alive.
  D5, `reed down`, D8's shield and the claim settle use that reading.
  For a dead batten, the claim settle names `lyx board set-status`.
- F8 (MEDIUM): fixed as proposed.
  D8's unregistered rule needs a deleted directory outside every worktree git still registers.
  The Bound and acceptance row gain the live-pair case.
- F9 (MEDIUM): fixed as proposed.
  D4's re-check refuses only for `RefuseWhenBusy` callers.
  Batten's teardown and the forced reconcile teardown go on and report `DriverWasLive`.
  Updated the acceptance row.
- F10 (LOW): the PATTERN-pair-teardown text now names "done locally with batten no longer running", and D5 ties that phrase to its two local readings.
- F11 (LOW): D10's Rationale cites the 45-minute Webster fork of 2026-10-14 from the `pause-background-work` note and the fifteen-minute margin.
- F12 (LOW): D8's reap ends unregistered orphans with D3's SIGTERM, five seconds, SIGKILL sequence.
- F13 (LOW): rewrote the D11 Open risks line as proposed.
- F14 (LOW): D4 and Open risks state the Windows `ErrDriverBusy` refusal for a paused run with a live Go driver, and give the way forward.
- F15 (LOW): renumbered the support log's (D7) to (D8) and D10's arrival rule to D11's.
  The `relevant.md` part is deferred (below).
- F16 (NIT): reworded the Constraints fact to `Resume` and `Lift` sharing `liftEntries`.

## Self-check

- fix-proposal, BLOCKING: a step waiting on its interrupted agent under a pause holds the run lock with no waiting marker, so remove of a paused run refused.
  Fixed: D5 also passes the holder under a covering pause when the run holds no parent lock record.
  Added a Constraints fact (`shedverbs/doc.go`) and updated the acceptance row.
- fix-claims, MEDIUM: the step child is a read descendant `End` signals, not outside it.
  Fixed: D3's line and the `StartTied` fact now name `Setpgid`.
- fix-way-forward, MEDIUM: on Windows `lyx tender status` lists no pid, so the `taskkill` way forward was unreachable.
  Fixed: the refusal names the pid from the owner record, and D8 lists registered records on every platform.
- fix-consistency, MEDIUM (merged with fix-proposal LOW, same point): "every reader" of the batten reading left D5 and `reed down` on the raw record.
  Fixed: both use D4's reading, and the D5 acceptance row was updated.
- fix-consistency, MEDIUM: `reed down`'s local owner verdict sat outside `PATTERN-origin-reconcile` and `PATTERN-owner-machine`.
  Fixed differently: `reed down` takes no owner verdict, since the owner pause's entry covers that case, and Constraints says why it runs no reconcile.
- fix-proposal, MEDIUM: D7 left a batten whose chain `End` ended with nothing to revive it once ownership returned.
  Fixed: D7 has batten treat that child as halted with a dead driver strand, and the acceptance row was updated.
- fix-claims, LOW: `DriverWasLive` is set today only by a spent quiet wait.
  Fixed: D4 says the re-check sets it.
- fix-way-forward, LOW: the run-lock refusal had no way forward.
  Fixed: it names retrying once the named pid ends, and the pause case removes the never-ending holder.
- fix-consistency, LOW: `PATTERN-pause-holds-prompts` did not mention the waiting marker.
  Fixed in Constraints.
- fix-consistency, LOW: Q16 was stale against D7.
  Fixed.

## Disputed

None.

## Deferred

- F15, the `relevant.md` part: `_lyx/discussion/relevant.md` is outside this round's write surface, which allows only `decision-record.md`, `support-log.md` and this report.
  Its lines "the Windows bound of D3 and D7", "hibernateengine … D9" and "shuttleengine … D10" still need renumbering to D8, D10 and D11.
