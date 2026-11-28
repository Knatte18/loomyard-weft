# darn: give measurement stops a way forward, bar subagents when gated

This closes two gaps in the rule for an entry whose deliverable is gate measurements.
The change is to the two darn Step 4 stencils, `darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`,
plus assertions in `TestDarnSpec`.

- **A way forward with no parent.**
  While the task's scope is open, a writer with no parent and no `Added after Discussion` entry allowing the calls
  still stops.
  Its stop report now names `lym loom decision add`, with an entry naming the allowed calls and their limits,
  then `lym loom resume`; the resumed writer runs the calls under that entry.
  Before, the report asked for a decision without saying how it reaches the run.
- **No subagents for measurement calls when gated.**
  The gated fragment now says the writer runs these calls in its own session and no subagent runs them,
  as the ungated fragment already did.

## Design decisions

- On a task that a Code-Review round has run on, the scope is closed
  and no decision allowing the measurement calls can be added.
  There the stop report instead names a board note titled `Decision for <slug>: <call>`,
  saying the scope is closed and filing the measurements for a new task, then `lym loom resume`.
- In both fragments, such a note settles the point for the resumed writer:
  it runs none of the calls, does not stop, makes what the record asks without them,
  and names the note in its change description's design decisions.
  Without this clause the resumed writer would stop again on every resume.
- The fragments spell the slug as `<slug>` in words, because they are not filled with a slug marker.

## To check by hand

- **Prose-only assertions** (`internal/darnengine/spec_test.go`, `measureNoSubagent` and `measureNoParentWayForward`):
  both fragments carry both lines, so the assertions tell neither fragment apart and pin only stencil wording, not behaviour.
  Decide whether to keep them as the test that fails without this fix, or to remove them.
- **Resume loop on a closed scope** (the no-parent branch of the measurement rule in both fragments):
  without a clause for the closed-scope note, a resumed writer stops again each time.
  The clause is in both fragments; read it and confirm it ends the loop.
  No resumed run was exercised.
- How the later checks treat a measurements-only task that such a note settles, with no commit on the branch, was not examined.

## Merge danger

**Door:** two-way
**Blast radius:** darn
