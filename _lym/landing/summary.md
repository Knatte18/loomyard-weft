# Have the darn writer ask before an entry's measurement gate calls

An entry can have gate measurements as its deliverable, such as test times per package, one `lym gate test` call per package and tier.
The darn writer's gated Step 4 list, `darn-fragment-make-change-gated.md`, forbade those calls and named no exception for them,
so a writer on it had to either leave the entry undone or break a stated rule on its own decision.
The other list, `darn-fragment-make-change.md`, had no rule or limit for them.

Both Step 4 fragments, `darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`, now carry a rule for these calls:

- They fall outside the other test-running rules of that step.
- Before any of them, the writer sends its parent every planned call and their count, and ends its turn to wait.
- It runs only what the answer allows, within the limits the answer sets, one at a time in the foreground.
- With no parent, or when the post is refused and no strand answers,
  it runs only what an `Added after Discussion` entry of the record allows.
- With no such entry it runs none, writes a stop report that asks for a decision on the measurement runs and their limits,
  creates the stop marker and makes no further commit.

In `darn-fragment-make-change.md`, where subagents run the gate themselves, the measurement calls stay in the writer's session,
and the gate run after each commit's last edit still follows.
In the gated fragment, the verify gate's test of the committed tree is not one of these calls and still runs.

The doc comments in `contracts/stencils/stencils.go` and `internal/darnengine/spec.go` state the new rule.
No Go behavior changes.

Closes #283

## To check by hand

- No-parent path and the entry's deliverable, in Step 4 item 3 of both fragments:
  with no parent and no `Added after Discussion` entry allowing the calls, the measurements are not produced.
  Check that the stop report and stop marker make this a visible stop, so the deliverable is not silently dropped.
- Subagents and the measurement rule, in Step 4 items 2 and 3 of the fragments:
  check that no subagent can make measurement calls without the question to the parent.
  `darn-fragment-make-change.md` says that no subagent runs them.
  `darn-fragment-make-change-gated.md` has no such sentence and relies on item 2, which tells each subagent to run no `lym gate test`.
- The new wording has no test, and the diff holds no run of a darn writer against a measurement entry.

## Merge danger

**Door:** two-way
**Blast radius:** darn
