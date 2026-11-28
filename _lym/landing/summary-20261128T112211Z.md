# Darn writer asks its parent before the gate calls of a measurement entry

The darn writer's Step 4 lists had no rule for an entry whose deliverable is itself gate measurements, such as test times per package and tier.
With a cold fixer on the verify gate, `darn-fragment-make-change-gated.md` forbade every `lym gate test` call bar one measuring run of one test class,
so a writer could either refuse the entry or break the rule on its own decision.
Without a cold fixer, `darn-fragment-make-change.md` allowed gate runs per commit but set no bound on a measurement series.

Both lists now carry the same rule in their item 3:
before any gate call for such an entry, the writer asks its parent with `lym creel post --parent --expect-reply`,
naming every planned call and their count, and ends its turn to wait.
It runs only the calls the answer allows, within the limits the answer sets, one at a time in the foreground.
With no parent it runs none of them and names them in the change description.

## Design decisions

- The rule asks the parent rather than granting the calls with fixed limits:
  the right number of calls, tiers and repeats depends on the entry, and the parent can weigh them against the hub's other runs.
- The rule sits in both lists, since an ungated writer meets the same entry and its per-commit gate rule does not bound a measurement series.

## Changes

- `contracts/stencils/darn/darn-fragment-make-change-gated.md` and `contracts/stencils/darn/darn-fragment-make-change.md`: the rule and their header comments.
- `contracts/stencils/stencils.go` and `internal/darnengine/spec.go`: doc comments name the rule.
- `internal/darnengine/spec_test.go`: `TestDarnSpec` checks that both the gated and the ungated prompt carry the rule.

## Merge danger

**Door:** two-way
**Blast radius:** darn
