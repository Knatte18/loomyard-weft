# Decision record — idle-notice-misses-fork-activity

## Goal

A live agent run that works through one long turn, its own transcript or its forks' transcripts being written, reads as active to batten's idle check, so no quiet notice goes while the writer works.

## Scope

- `shuttleengine.ReadAgentActivity`'s per-run reading, in `internal/shuttleengine/activity.go`.
- Its test table in `internal/shuttleengine/activity_test.go` and the reader's paragraph in `internal/shuttleengine/doc.go`.
- Out of scope: batten's notice logic in `internal/battenshed`, which judges whatever readings it is handed, and the Claude engine's transcript and fork reading, which already counts forks.

## Decisions

### Read the session's activity before the run's first own turn end

Decision: when a live run's events file holds no turn end past its prompt offset, `ReadAgentActivity` hands the newest turn end before the offset to the engine's `ActivityReader` and takes only its last-activity time, forks included.
The API-error part of that reading is dropped, so a turn end before the offset still never reports an API error for the run.

Rationale: the code already counts fork transcripts, but only once a turn end past the prompt offset names the transcript.
A writer whose task is one long turn, delegating to a fork, has no such turn end until it finishes, so its transcript and forks were never read and the run read idle since its skill-load turn.
The skill-load turn end before the offset belongs to the same session and names the same transcript.

Bound: only the last-activity time is taken from a turn end before the offset; it can only make a run read more recently active, never raise a notice or an API error.
A run with no turn end at all in its events file keeps today's reading.

## Constraints

- `PATTERN-shuttle-provider-seam`: the transcript and fork layout stays inside `internal/shuttleengine/claudeengine`; the change sits in the provider-neutral reader.
- `PATTERN-test-economy`: the behavior is a new row in `TestReadAgentActivity`, not a new test.
- `PATTERN-comment-line-breaks`: one sentence per line in every comment touched.

## Auto-mode assumptions

- The idle notice the board entry cites came from this gap: the run ran on another machine and its logs are not readable here, so the cause is inferred from the code.
- A lyx-spawned session's events file holds a turn end before the prompt offset whenever its skills were loaded in a turn of their own.

## Open risks

- The board entry quotes the notice as "no agent activity readable", the wording batten uses when it finds no live run at all, while this gap yields "agents idle".
  If the quote is exact, a second cause hides a live run from the reader, and this change does not address it.
- A session that reached its prompt with no turn end before it, such as one launched with no skills, stays unread until its first own turn end.

## Acceptance criteria

- A live run whose only turn end lies before its prompt offset reports the session's last activity, forks included, when that is newer than its events file.
- That run reports no API error even when the reading of the turn end before the offset carries one.
- A run with a turn end past its prompt offset reads as before, the pre-offset turn ends ignored.
- A run with no turn end at all is judged by its events file alone, as before.
