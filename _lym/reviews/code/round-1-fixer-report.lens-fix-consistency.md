```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F4 change set against its surroundings.

1. **LOW**: `contracts/stencils/darn/darn-fragment-make-change.md` and `-gated.md` now scope the parent-first rule to the measurement calls alone, but the prose describing that rule still says "the gate calls of an entry".
   - The twins are:
     - the ungated header: "the writer and its subagents run the gate themselves, bar the gate calls of an entry whose deliverable is gate measurements"
     - the gated header: "and for the gate calls of an entry … before it runs any"
     - both fragment doc comments in `contracts/stencils/stencils.go` ("before the gate calls of an entry …")
     - the `DarnTable` doc in `internal/darnengine/spec.go` (same phrase)
   - The ungated header is now wrong: the fragment says "The gate run after the last edit of each commit still follows for the entry's code or committed results", and that run is a gate call of such an entry.
   - Fix by writing "the measurement calls" in each place.
   - None of these mentions the new stop with no parent either, but they stay true without it.

2. **LOW**: in both fragments, the limit "one at a time in the foreground" is tied to the parent's-answer sentence ("Run only the calls your parent's answer allows, within the limits it sets, one at a time in the foreground"). The new path, "run only the calls an `Added after Discussion` entry of the record allows, within its limits", does not carry it. A writer with no parent, resumed with such an entry, could read the calls as free to run in parallel or in the background. The session-wide gate rule may already cover this.

Otherwise these agree:
- "The stop marker Step 3 names" resolves to `darn-template-write.md` Step 3.
- `darnwrite.go` halts Stuck on that marker whether the session ends Done or gate-failed, and appends a `lym loom decision add`, then `resume`, way forward. That matches the `Added after Discussion` re-entry.
- The refused or unanswered post clause matches `parent-directive-parent.md`.
- The gated item 2 already bars subagents from running gate calls, so it needs no twin of the ungated "Keep here too" line.
- `spec_test.go` has no remaining reference to the deleted constant, going by the edits shown; the gate run passed.

Tool calls: none.
