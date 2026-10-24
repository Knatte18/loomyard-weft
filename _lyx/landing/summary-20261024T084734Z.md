# Shuttle sends no hold notice for a turn end the usage limit stopped

When the account hits its usage limit, an autonomous agent ends its turn without its output files and shuttle holds it.
Until now the hold also sent the parent a notice, which the orch cannot act on while the account is limited.
The notice only lands in the orch's context after the reset, as stale clutter.

A held turn end that the provider's usage limit caused now sends no hold notice, deferred or not, and leaves `RunState.NotifiedOffset` alone.
The hold, the wait mark, the run's deadline and the API-error continue after a machine sleep are unchanged.
The next held turn end that is not limited is notified as before.

## How the limit is recognised

`internal/shuttleengine` gains the optional `UsageLimitReader` capability beside `Engine`.
Wait asks it once per held turn end, through the turn end's events-file line, which `heldTurnEnd` now keeps.
An engine without it reports no limit.

Claude implements it in `internal/shuttleengine/claudeengine/activity.go`, reading Claude Code's own error classification, `error: "rate_limit"` on a synthetic `isApiErrorMessage` transcript entry:

- the main transcript's newest main-chain turn end is such an entry, or
- a fork transcript under `subagents/`, last written after the main chain's previous turn end, has such an entry as its newest assistant entry.

The fork half covers the case the board entry reports: forks hit the limit and the agent then ends its own turn with a message saying so.
The time bound keeps a limit an earlier turn already met from silencing a later notice.

## Design decisions

- Recognition is a separate capability, not a field on `TurnEndActivity`: that reading runs on every session-state read, and finding the previous turn end can walk far back in a long transcript.
- The session-state cause stays `api-error` for a main-chain limit, so `lyx revive` still reaches a limited session after the reset.
- The classification field is read, never the error text or the agent's words: the text changes between Claude Code versions, and the notice treats the agent's words as a report.
- A transient API rate limit carries the same classification and is skipped too; for a subscription session it is the usage limit in practice.
- An agent that stops on its own about a limit, with no limit recorded in its main chain or a fork, is still notified.

## Tests

- `TestWait_HeldTurnEndNotice` gains rows for a usage-limited stop (no notice, offset untouched, held to the deadline), a later unlimited stop (notified), and a deferred run (no notice after the deferral either).
- `TestTurnEndUsageLimited` covers the main-chain limit, a server error, a plain turn end, a fork limited during and before the turn, a fork that answered after a limit entry, a first turn, and a missing transcript.

Docs: `internal/shuttleengine/doc.go` and `internal/shuttleengine/claudeengine/doc.go`.
