# Count a session's fork activity before a run's first own turn end

Batten sent the orch an idle notice for a run whose Plan writer was busy in a fork.
The writer's task was one long turn that delegated to a fork, so its events file held no turn end past the prompt offset until it finished.
`shuttleengine.ReadAgentActivity` hands a turn end to the engine's `ActivityReader`, which already reads the session's transcript and its forks, but only a turn end past the offset was handed over.
Until then the run read idle since its skill-load turn.

Now, when a live run has no turn end past its prompt offset, `ReadAgentActivity` hands over the newest turn end before the offset.
That turn end belongs to the same session and names the same transcript, so a writer working in a fork reads active and raises no notice.

The reader's paragraph in `internal/shuttleengine/doc.go` is updated to match.

## Design decisions

- Only the last-activity time is taken from a turn end before the offset.
  Its API error is dropped, so such a turn end can make a run read more recently active but never raises an API error or a notice.
- A turn end past the offset still wins, and the turn ends before the offset are then ignored.
- A run with no turn end at all, or an engine without `ActivityReader`, is judged by its events file alone, as before.
- The change sits in the provider-neutral reader, so the transcript and fork layout stays inside `internal/shuttleengine/claudeengine`.
- `newestTurnEnd` parses the two parts of the events file separately and logs the parse recovery once per successful path, so a part that fails to parse cannot end and restart the logging episode on each poll.
- The behavior is a new row in `TestReadAgentActivity`.

## Check by hand

- The board entry quotes the notice as "no agent activity readable", the wording batten uses when it finds no live run at all, while this gap yields "agents idle".
  If the quote is exact, a second cause hides a live run from the reader, and this change does not address it.
  The run ran on another machine, so its logs were not available to confirm which wording it showed.
- A session that reached its prompt with no turn end before it, such as one launched with no skills, stays unread until its first own turn end.
