# Darn measurement rule: a no-parent way forward and a subagent bar

The darn writer's Step 4 lists carry a rule for an entry whose deliverable is gate measurements, such as test times per package:
the writer asks its parent before any of those calls and runs only what the answer allows.
Two gaps remained in that rule.

## No-parent path

With no parent, and no `Added after Discussion` entry allowing the calls, the writer stopped
and named as the way forward only "a decision on the measurement runs and their limits".
Nothing said how that decision reaches the run, so the measurements the entry asks for had no route to being made.

Both fragments, `contracts/stencils/darn/darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`, now have the stop report name:

- `lym loom decision add` with an entry naming the allowed calls and their limits, then `lym loom resume`,
  after which the writer runs them under that entry, as the rule's existing no-parent branch already allows;
- on a task whose scope a Code-Review round has closed, where the record takes no new entry,
  a board note filing the measurements for a new task, then `lym loom resume`.

This mirrors the way forward the darn template's Step 3 gives for an outdated entry.

## Subagents in the gated list

`darn-fragment-make-change-gated.md` relied on its item 2, which tells each subagent to run no `lym gate test`,
and had no sentence of its own keeping measurement calls out of subagents.
It now says, as the ungated list already did: "Run them in this session: no subagent runs them."

## Tests

`TestDarnSpec` in `internal/darnengine/spec_test.go` checks both lines in the rendered prompt of its ungated and gated rows.
On the merge base the gated row fails on both lines and the ungated row on the way-forward line.

## Design decisions

- The closed-scope case gets its own way forward because a record entry cannot be added once Code-Review has run,
  so naming only `lym loom decision add` would point at a refused step.
- The doc comments in `contracts/stencils/stencils.go` and `internal/darnengine/spec.go` stay as they are:
  they state that the writer asks before measurement calls, which still holds, and name neither changed detail.

## Merge danger

**Door:** two-way
**Blast radius:** darn
