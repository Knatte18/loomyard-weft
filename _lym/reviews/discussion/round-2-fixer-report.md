# Fixer report — discussion round 2

## Changes

- F1, MEDIUM, fixed differently.
  The fact is settled from Claude Code 2.1.312's bundle: the task-notification row component (`Rp`/`kp`) draws the summary, status, duration and a background command's delivery note, and a body only for a monitor event, so the terminal never draws the wake's body.
  D4's rationale records this evidence, the open risk is cut, the D3 real-session criterion now expects one `creel mail` row with no body and the mail once as the echo, and Q11 is resolved.
  Neither proposed operator copy works: a per-wake `rewakeSummary` on the hook's stdout is honored only when `rRe(pluginId)` passes the Anthropic-marketplace set `A5e`, and lym's waiter is a settings hook with no plugin id; the settings label is fixed for the session.
  New D6 gives the operator the copy instead: the daemon keeps an acknowledged ring's lines and ids as the strand's echo, the flag stands while it is owed, and the strand's next claim hands it out ahead of its mail as `systemMessage` under `[creel] Delivered while idle:`.
  D1, D2, D5, Scope, Constraints, the acceptance criteria and the Notes follow, and the support log adds Q12 and the rejected alternatives.
- F2, LOW, fixed as proposed, taking the switch: `hook-wait` exits the wake code once the acknowledgement is written, whether it is confirmed, refused or the confirmation is lost; exit 0 is kept for a failed write of the lines or the acknowledgement.
  D3 states the loss-against-duplicate choice, its Bound now names the duplicate, the Constraints exception and the D3 hook-wait criterion follow, and the exit-0 policy is a rejected alternative.
- F3, LOW, fixed as proposed: D3 keeps a ring outcome that reached its handler off the client's reconnect-and-resume path, the Notes name `Client.CallWatching`'s resume of `VerbHookWait` after a `connLostError`, and Scope names the `internal/creelengine/doc.go` sentence on resuming a dropped hook-wait.
- F4, NIT, fixed as proposed: D5 adds `ack-failed` at `Warn`, replacing the daemon's journal-failure `Warn` for a claim, and its acceptance bullet names it.

## Self-check

- fix-proposal, fix-way-forward and fix-claims, MEDIUM, merged: a refused acknowledgement leaves the ids after the failing one reserved (`Reading.Ack` returns early, `handOut` never releases), so the promised duplicate never happens. Fixed: D3 releases every id an acknowledgement's failed journal write left unjournaled, on every claim path, with an acceptance bullet; D5's `ack-failed` names the released ids; the Constraints exception points at D3's rule.
- fix-consistency, MEDIUM, and fix-way-forward, LOW, merged: a ring woken by a refused or released acknowledgement stays unanswered, so the bell may follow. Fixed: D3 and its Bound state it.
- fix-consistency, LOW: D5 named both `released` and `ack-failed` for a ring journal failure. Fixed: `released` now covers ends before the client's acknowledgement reaches the daemon.
- fix-consistency, LOW: D6 kept lines only while D5 logs echo ids. Fixed: D6 keeps the lines and their ids.
- fix-claims, MEDIUM: a woken turn an API error ends fires `StopFailure`, not `Stop`. Fixed: D6's rationale drops "every woken turn ends at a Stop", and its Bound states the `StopFailure` case.

## Disputed

None.

## Deferred

None.
