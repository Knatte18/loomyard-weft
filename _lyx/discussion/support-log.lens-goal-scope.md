```yaml
lens: goal-scope
outcome: reported
```

Lens: goal-scope. I checked the decision record against its board entry for missing asks, unlicensed additions, hidden deferrals and whether this is the right thing to build.

**MEDIUM — D1 / Auto-mode assumptions: the record settles a plainly outdated entry itself, against its own D9 rule.**
The board entry says to "keep it at `pr-gate`, where it approves the landing".
D1 instead deletes the alignment agent completely and lists that only as an auto-pick.
By the record's own definition (D6/D9), an entry overtaken by landed work (`fc75aef79`) is a question only the parent can settle, and the writer must record it under `## Open risks` rather than settle it.
The operator may still want a landing-time approval, for example by folding it into the Parent-Align fixer, or may not.
The record should name this under `## Open risks` as a parent question, whatever D1 keeps as the default.
Evidence:
- Board entry Part 1 Design: "keep it at `pr-gate`".
- `internal/alignment/doc.go`: "The agent runs at one point of a run: `parent-review`".
- D9's own rule.

**MEDIUM — D6 / D5: `board-fit` in the perch has 6 tool calls, but D6 asks it to read more than that.**
Every perch fork is capped at 6 tool calls and is told to work from the reading manifest.
D6 asks for:
- `lyx board list`
- `lyx board get` for every entry that touches the task
- `lyx loom peers`
- `lyx loom peers show` per close peer
- the diffs of landed parent commits

The alignment agent it replaces had no such cap, so the record does not show the perch can do the check the entry moves into it.
The record needs to say how the sources fit, for example by adding the board list and `lyx loom peers` output to the round's reading manifest, or by stating the cap as a bound in D6.
Evidence: `contracts/stencils/burler/burler-cluster-fan.md:16`: "You may make at most 6 tool calls, each only for something your procedure needs that the manifest lacks".

**LOW — D8: suppressing the fix row's notice in the Discussion segment changes behaviour the entry did not ask for.**
The entry says "what is left is the Bouncer's routing".
Silencing the `2e265a4fc` notice is an extra change.
The reason given (one message per round) is sound, but D8 should state it as a deliberate change to `2e265a4fc`'s behaviour, and the acceptance criteria should pin it, which they already do.
Evidence: board entry Part 1, bullet 2.

**LOW — D4: keeping a legacy parser in `reportengine` is an addition the entry did not ask for.**
It follows from D1 and D2 and is justified by audit continuity.
It is new carrying cost, though, so it should stay a recorded auto-pick, which it is.
Evidence: D4; Auto-mode assumptions.

**LOW — D10: the record does not say what the run does when the darn writer's wait times out.**
The wait is bounded by `writer_timeout_min`, but the record never says what that bound ends in: a halt, a hand-back, or a respawn that re-checks.
The entry says "asks the parent and waits instead of guessing", and a timeout path that respawns would skip the first-spawn check and could then guess.
Evidence: D10 says the check is taken "only on a first spawn (prior work `(none)`)".

The rest holds:
- All three parts are covered.
- The fabric merge-in and the merge record are explicitly out of scope, with reasons.
- No MVP phase or deferral hides in the text.
- Notes, not tasks, are left for the orch to rewrite after landing.
- The rename's list of keys is complete: `merge_fix_attempts` and the `merge-in` gate key; the fixer role is `fixer`, so it needs no rename.

Tool calls:
- grep of the burler stencils for "tool call": my context lacked the perch fork's tool-call cap, which bears on whether `board-fit` can do D6.
- grep of `landingshed` for the fixer role name: my context lacked whether a session role carries "merge", which bears on the completeness of D11's rename list.
