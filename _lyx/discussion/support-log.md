# Support log — usage-limit-hold-quiet

## Interview

A darn run holds no interview; the board entry and a coarse reading of `internal/shuttleengine` and `internal/shuttleengine/claudeengine` are the inputs.

## Rejected alternatives

- A new session-state cause for a usage limit: `lyx revive` and the conditioned sends match cause `api-error`, so a new cause would stop revive from reaching a limited session.
- Detecting the limit inside `TurnEndActivity`: that reading runs on every session-state read, and finding the previous turn end can walk far back in a long transcript.
- Matching the agent's last message for limit wording: the notice treats that text as a report and not an instruction, and an agent's words are no reliable signal.
- Matching Claude Code's limit text in the transcript or the pane: the wording changes between versions, while the `error` classification is machine-set.
- Deferring the notice until the limit resets: the reset time is only in the error text, and parsing it would be fragile.

## Review rounds

_No rounds yet._

## Question ledger

- Should a limited turn end be resumed automatically after the reset? Self-pick: no; out of scope, `lyx revive` and SendMessage already exist.
- Should a fork's limit count, or only the main chain's? Self-pick: both; the incident's agent ended its own turn, which only a fork's limit explains.
- How is "during the turn" bounded for a fork? Self-pick: last written after the previous main-chain turn end.
