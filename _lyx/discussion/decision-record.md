# Decision record — usage-limit-hold-quiet

## Goal

A held turn end that the provider's usage limit caused sends no hold notice to the parent, so the orch's context is not cluttered with notices it cannot act on while the account is limited.

## Scope

- In: recognising, at a held turn end of a shuttle run, that the provider's usage limit stopped the session or one of its forks during the turn, and skipping that turn end's hold notice.
- In: the provider-neutral capability in `internal/shuttleengine` and Claude's implementation in `internal/shuttleengine/claudeengine`.
- In: the package docs of both packages.
- Out: resuming a limited session once the limit resets; `lyx revive` and an operator's SendMessage stay the way forward.
- Out: the dialog notice, the API-error continue after a machine sleep and the hold itself, which are unchanged.
- Out: reading the agent's own words for a limit.

## Decisions

### Recognition is an optional engine capability read once per held turn end

Decision: `internal/shuttleengine` declares an optional capability beside `Engine` that reports whether the provider's usage limit stopped the session a turn end names, or one of its forks during that turn.
Wait asks it once for each held turn end, and an engine without it reports no limit.
Rationale: the provider-seam invariant keeps the transcript shape in the engine package, and asking only at a held turn end keeps the cost off the hot session-state reads.

### Claude reads the limit from Claude Code's own error classification

Decision: Claude's implementation reports a limit when the newest main-chain turn end of the transcript is Claude Code's synthetic API-error entry with `error` set to `rate_limit`,
or when a fork transcript beside it last written after the previous main-chain turn end has, as its newest assistant entry, such an entry.
Rationale: the `error` field is Claude Code's machine classification, also carried by the `StopFailure` hook, and stays stable where the error text changes between versions.
The fork half covers a parent agent whose forks hit the limit and that then ends its own turn, which is the case the board entry's incident shows.
Bound: a fork counts only when written during the turn, so a limit an earlier turn already reported never silences a later notice.

### A usage-limited held turn end sends no notice and defers none

Decision: Wait logs a usage-limited held turn end at Info and sends no hold notice for it, deferred or not; the hold, the wait mark, the deadline and the API-error continue after a sleep are unchanged.
RunState.NotifiedOffset is left alone.
Rationale: the parent cannot act while the account is limited, and a notice queued then lands in its context after the reset as stale clutter.
Bound: the skip covers only the turn end the capability reports limited; the next held turn end that is not limited is notified as before.

## Constraints

- `PATTERN-shuttle-provider-seam`: the `rate_limit` literal and the transcript shape stay in `internal/shuttleengine/claudeengine`.
- `PATTERN-completion-signal`: the output-file check still runs before a turn end is held; nothing here changes it.
- `PATTERN-test-speed` and `PATTERN-test-economy`: rows in the existing hold-notice and activity tests, fake clocks, no real transcript outside `t.TempDir()`.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- Claude Code marks a subscription usage limit with `error: "rate_limit"` on its synthetic API-error transcript entry, in the main chain and in fork transcripts alike.
- A transient API rate limit reads the same and is also skipped; for a subscription session it is the usage limit in practice.

## Open risks

- A session whose agent stops on its own, mentioning a limit, with no limit recorded by the provider in the main chain or a fork, is still notified.
- A Claude Code release that renames the `rate_limit` classification silently restores the notices.

## Acceptance criteria

- A held turn end whose engine reports a usage limit sends no notice, leaves NotifiedOffset at zero and holds to the deadline.
- A later held turn end the engine does not report limited is notified.
- A deferred run with a usage-limited held turn end sends no notice after the deferral either.
- Claude reports a limit for a main-chain `rate_limit` API-error turn end and for a fork stopped on `rate_limit` after the previous turn end.
- Claude reports no limit for a `server_error` turn end, for a plain turn end, and for a fork stopped on `rate_limit` before the previous turn end.
- An unreadable or missing transcript reports no limit.
