# Plan-Burler round 1 — fixer report

## Changes

- **F1 (LOW, scope)**: fixed. The fix as proposed is applied, plus one change to a target so the file header is in scope.
  Premise checked: `internal/shuttleengine/activity.go:19` reads "TurnEndActivity returns the activity of the transcript turnEnd names.";
  `internal/shuttleengine/claudeengine/activity.go:22` reads "Its modification time is the last write time";
  `claudeengine/activity.go:1` reads "the transcript a Stop payload names through transcript_path is read for its last write time".
  After card 2, all three are wrong for a session with fork activity.
  In `_lyx/plan/02-fork-activity.md`:
  - `**Edit:**` gains `internal/shuttleengine#ActivityReader`, as proposed.
  - The Docs paragraph now requires the `ActivityReader.TurnEndActivity` interface method doc, the `ActivityReader` and `TurnEndActivity` type docs, the `Claude.TurnEndActivity` doc and the `claudeengine/activity.go` file header to say the reading is the session's last activity with forks included.
    They also say a fork's pending call reads active up to `bashMaxTimeout` past its last write, and that the API-error part reads only the main transcript.
    The proposal did not name the two type docs (`ActivityReader`: "the provider's reading of the transcript a turn end names"; `TurnEndActivity`: "reading of a session's transcript at a turn end").
    They state the same old contract and sit inside targets the card already lists, so they are covered too.
  - Not in the proposal: `internal/shuttleengine/claudeengine#Claude.TurnEndActivity` is replaced by the whole-file target `internal/shuttleengine/claudeengine/activity.go`.
    A member glyph's span does not cover the file header the fix must change.
    Listing the file beside the member is refused by the spec's `redundant-file-target` check (row 33), whose way forward is to "keep the file when the card changes the whole file".
    No other card targets a member in that file.
    The Intent records why the target is a whole file.

`lyx loom validate-plan` passes on the edited plan (`"ok":true`).

## Disputed

None.

## Deferred

None.
