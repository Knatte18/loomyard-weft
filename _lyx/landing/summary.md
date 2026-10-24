# shuttle: send no hold notice for a turn end the usage limit stopped

A held turn end that the provider's usage limit caused no longer sends a hold notice to the parent.
The parent cannot act while the account is limited, and a notice sent then only lands in the orch's context as stale clutter after the reset.
The hold, the wait mark and the deadline are unchanged, so `lyx revive` or an operator's `SendMessage` stays the way forward.

Closes #854

## Design

- `internal/shuttleengine` declares an optional `UsageLimitReader` capability beside `Engine`.
  `Wait` asks it once for a held turn end that would otherwise be notified or queued for notice.
  An engine without it reports no limit, so its turn ends are notified as before.
- A limited turn end is logged at Info and sends no notice, deferred or not.
  `RunState.NotifiedOffset` is left alone, and the next held turn end that is not limited is notified as any other.
- The check runs only where a notice would otherwise go, through `holdNoticePending`.
  It skips runs `notifyHeld` never notifies, a deferred-hold run whose next turn already started, and a non-deferred run covered by a pause, which discards the notice anyway.
  A covered deferred-hold run still asks, since its queued hold is sent after the pause lifts.
- `heldTurnEnd` now carries the turn end's raw events line, which is what the capability reads.
- `claudeengine` reports a limit in two cases, both read from Claude Code's own `error: "rate_limit"` classification on its synthetic API-error transcript entry:
  - the newest main-chain turn end is such an entry;
  - a fork transcript beside the main one, written after the previous main-chain turn end, has such an entry as its newest assistant entry.
  The fork case covers a parent whose forks hit the limit and that then ends its own turn, which is the incident in the issue.
  The time bound keeps a limit an earlier turn already reported from silencing a later notice.
- An unreadable or missing transcript reports no limit.
- The `rate_limit` literal and the transcript shape stay in `claudeengine`, per the provider-seam invariant.
  Both packages' `doc.go` describe the behavior.

## Known gaps

- A transient API rate limit reads the same as the usage limit and is also skipped; for a subscription session it is the usage limit in practice.
- An agent that stops on its own while mentioning a limit, with nothing recorded by the provider, is still notified; the agent's words are deliberately not read.
- A Claude Code release that renames the `rate_limit` classification silently restores the notices.

## Check by hand

Not run against a live limited session: confirm that a real subscription limit writes `error: "rate_limit"` in both the main and a fork transcript, and that the orch receives no hold notice.
