# Support log — idle-notice-misses-fork-activity

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Add fork reading to batten's idle check, as the board entry's idea puts it: rejected, since `claudeengine`'s `TurnEndActivity` already reads fork transcripts under `subagents/`; the gap is that it is never called before the run's first own turn end.
- Locate the transcript from the run's session id and pane cwd instead of a turn end: rejected, since `ReadAgentActivity` is a files-only reader with no pane cwd and widening the `ActivityReader` seam costs more than the gap needs.
- Take the whole reading, API error included, from the turn end before the offset: rejected, since an existing row pins that a skill-load turn end's API error is not the run's own.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does batten's idle check miss fork activity? Self-pick: no; forks are counted whenever a turn end past the prompt offset exists, and missed before one exists. Resolved.
- Q: Can the incident's own logs confirm the cause? Self-pick: no; the run ran on another machine and its archived records hold no step history, so the cause is inferred from the code and the mismatch in notice wording is an open risk. Resolved.
- Q: Which turn end stands in before the first own one? Self-pick: the newest turn end before the prompt offset, for its activity time only. Resolved.
