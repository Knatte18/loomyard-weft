# Fixer report — discussion round 3

## Changes

- F1 [BLOCKING], fixed differently.
  The ids now live in a per-hub setup file under the hub's `.lym/slack/` directory, written only by `lym slack setup` after its `conversations.info` and `users.info` checks, through `internal/state`, owner-only.
  The `slack` config module keeps only `remind_after_min`.
  Changed in D2: the heading, the setup bullet, the per-poll read, a rationale sentence naming `hubWriteRefusal`, and the second Bound line, which now states the true residual of a direct file write or a process outside every pane.
  D3's start condition, its `lym slack status` report and its missing-setup refusal now name the setup file.
  The D2 and D3 acceptance bullets were updated, including that a `lym config slack` write does not repoint a live relay.
  Not made: the proposal to keep the ids in `slack.json` beside the token.
  That file is one per machine, since D1 has one app per machine, but D1 gives each hub its own channel, so an unkeyed `slack.json` cannot hold a hub's channel.
  D3 already puts per-hub state in the hub's `.lym/slack/`, so the per-hub setup file goes there.
- F2 [MEDIUM], fixed as proposed.
  D6's pane-ancestry bullet now defines the refusal of a caller in no known pane while any live strand's pane process is unknown, or the hub's strands cannot be read, matching `paneOfPeer` and `resolveSender` in `internal/creelengine/identity.go`.
  D2's refusal bullet and D3's `lym slack start` bullet refer to that check.
  The D2, D3 and D6 acceptance bullets include the unknown-pane case.
- F3 [MEDIUM], fixed differently in part.
  D3 states that the first read comes before any Slack poll and that every refused read is logged in the log `lym slack away` names.
  The relay ends, releasing its lock, only when creel refuses a first read because the relay runs in a non-orch strand's pane.
  Any other refused first read, such as while a pane process is unknown or the daemon is down, is retried with the poll's backoff, with the relay holding its lock and polling no Slack.
  D9 gives `relay` the `internal` audience, which `cmd/lym/clitree_test.go` requires of a hidden verb, and no operator audience.
  The D3 acceptance bullet was extended as proposed.
  Not made: ending the relay on every refused first read.
  The self-check showed that this leaves a relay started at a prime anchor change, while a pane process is unknown and presence is `terminal`, down with nothing to restart it, which breaks D3's acceptance that the relay is live after an anchor change.
- `support-log.md`: a Round 3 entry, and a note on the Turn 2 line that F1 moved the ids.

## Self-check

- fix-consistency, MEDIUM: the unknown-pane refusal named the operator's terminal, which that refusal itself refuses. Fixed with my own change: the refusal now names only waiting and running the verb again.
- fix-consistency, LOW: D2's Bound line cross-referenced D6 for direct file writes, which D6 does not state. Fixed with my own change: the cross-reference was dropped and the bound restated as a mistaken-run guard.
- fix-consistency, LOW: the D3 acceptance row lacked the unknown-pane case for `start`. Fixed as proposed.
- fix-consistency, LOW: support-log Turn 2 still said ids in machine config. Fixed as proposed: a round-3 note was added on that line.
- fix-proposal, MEDIUM, merged with fix-way-forward MEDIUM: ending the relay on any refused first read takes down a legitimately started relay that nothing restarts. Fixed with my own change: only a refusal for running in a non-orch strand's pane ends it, and every other refusal is retried holding the lock. A D3 acceptance row was added.
- fix-proposal, MEDIUM: the unknown-pane refusal can block `stop` and `back` from the terminal while a pane process stays unknown. Fixed with my own change: a D6 Bound line states the window and the Slack texts that still work. The fail-closed choice F2 asked for is kept.
- fix-proposal, LOW: D2's rationale, "such a write would skip setup's checks", held equally for a direct write to the setup file. Fixed as proposed: the rationale is scoped to lym's own paths.
- fix-claims, LOW: the check left out creel's second unknown case, when the hub's strands cannot be read. Fixed as proposed: D6 names it.
- fix-way-forward, MEDIUM: duplicate of the fix-proposal relay-ending finding, handled there.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
