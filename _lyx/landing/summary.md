# Read a run's session activity before its first own turn end

`shuttleengine.ReadAgentActivity` now reads a live run's session transcript, forks included, before the run's first turn past its prompt offset ends.
Batten's idle check and loom's loop watchdog both read agent activity through it, so a writer whose task is one long turn spent in a fork no longer reads idle since its start.

## The defect

The Claude engine's `TurnEndActivity` already counts fork transcripts under `subagents/`, but `ReadAgentActivity` called it only with a turn end past the prompt offset, since that turn end's `transcript_path` is how the transcript is found.
A writer that delegates its work to a fork ends no turn until the work is done, so until then its transcript and forks went unread.
Its activity fell back to the events file, whose newest Stop is the skill-load turn before the prompt, and the run read idle from its start.

## The change

- `newestTurnEnd` in `internal/shuttleengine/activity.go` returns the newest turn end past the prompt offset, and, when there is none, the newest one before it, flagged as not the run's own.
- `readRunActivity` hands either to the engine's `ActivityReader` and takes the session's last activity from it, but takes the API-error reading only from a turn end of the run's own.
  A skill-load turn's API error therefore still never reports for the run, as the existing row "turn ends before the prompt offset are not the run's own" pins.
- A run with no turn end at all in its events file keeps its events-file reading.

A new row in `TestReadAgentActivity` pins that a run whose only turn end lies before its prompt offset reports the transcript's activity and no API error.
The `ReadAgentActivity` doc comment and the agent-activity paragraph in `internal/shuttleengine/doc.go` describe the fallback.

## Design decisions

- The fallback uses the turn end before the offset rather than locating the transcript from the session id and pane cwd: the reader reads files only and is handed no pane cwd, and that turn end names the same session's transcript.
- Only the last-activity time crosses over from a turn end before the offset, so the fallback can make a run read more recently active and never raises a notice or an API error.

## Open risk

The board entry quotes the notice as "no agent activity readable", the wording batten uses when it finds no live run at all, while the gap fixed here yields "agents idle".
The incident ran on another machine whose logs are not readable here, so if the quote is exact, a second cause hid the live run from the reader and is not addressed by this change.
A session that reaches its prompt with no turn end before it, such as one launched with no skills, also stays unread until its first own turn end.
