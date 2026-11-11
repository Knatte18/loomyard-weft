# Refuse a raddle note dependency early; show a dead loop's own output

A raddle proposal whose part depended on a board note passed `raddle-propose` and was put to the operator.
Raddle-File then failed at the board write, because the board refuses a dependency on a note.
Separately, a `lym shed step --until-stop` loop that ended without an envelope reported a `loop log:` excerpt holding the envelopes of steps that had already finished, and `loop exit: not recorded` with no reason.
Landing d1535412 already records the loop process's exit status or signal; this change covers what that landing left.

## What changes

- `internal/raddleparser` has a new check, `depends-note`.
  It reports a part whose `depends_on` names a note on the board.
  The way forward is to drop the dependency, or to have the operator promote the note with the full `lym board promote '{"slug":"<note>"}'` command, then run `lym loom raddle-propose` again.
  The check runs wherever the raddle checks run: the Raddle-Write gate, `raddle-propose`, `raddle-approve` and Raddle-File.
  At Raddle-File it now sends the proposal back to Raddle-Write before the board write.
  Before, the board refusal surfaced as a failure the halt called transient.
  `contracts/specs/raddle-proposal-spec.md` lists the check.
- The board's own refusal of a dependency on a note now spells both commands with their arguments:
  `lym board promote '{"slug":"<note>"}'`, and `lym board set-deps` with the entry's slug and its remaining dependencies.
  Before, it named the bare command words in quotes.
- In `internal/shedverbs`, the loop writes a finished child's stdout to the loop log with each line opened by `lym shed: step <trace id> output: `.
  The dead-loop stop's `loop log:` excerpt leaves those lines out, so it carries only the loop's own output, such as a crash.
- An unrecorded loop exit now says why.
  When the log holds no start mark of the loop, the clause names that.
  When it holds one but no process marked the exit, it says no process marked it, as after a kill no process can catch or a stop of the machine.
- The `shedverbs` package doc follows both changes.

## Design decisions

- The note dependency is caught in the shared proposal checks rather than in `raddle-propose` alone.
  That way the gate, the approval and the filing all agree, and a proposal that slipped past an earlier check is still sent back before the board write.
- A dependency that names both a part and a board note counts as the part.
  The slug rules already refuse a part slug that is taken on the board.
- A child's stdout is prefixed line by line, not fenced by begin and end marks.
  Each line can then be classified alone, and a child line is never mistaken for a mark of the loop.
- When the loop has no output of its own, the stop carries no `loop log:` clause, as before, rather than a clause saying it is empty.

## To check by hand

- The note reported the board refusal as ending in `promote "testmap-upkeep" with ;`.
  The archived trace of that run holds the full text, with the command word in quotes.
  The command was most likely lost where the text was copied into the note.
  The change spells the command out in full either way.
- A loop log written before this change holds unprefixed child envelopes, so a dead-loop stop read over such a log still excerpts them.
