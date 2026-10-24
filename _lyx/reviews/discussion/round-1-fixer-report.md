# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed, the gap-and-bound branch: D5 moves every Go package load that lists test files, `impactset`'s graph and guard listing, `planglyph`'s type load, `cmd/testtiming` and `planparser`, to the tier tags.
  D13 states that the installed `lyx`'s narrowed gates, `--base` and computed, miss a package reached only through a `tier0` test import, and that the whole gates (Webster's end gate, Merge-In's `--all`, `main`'s `--upto 2`) bound it.
  The Constraint's "right for old and new trees alike" now applies only to the tier commands and points at D13.
  New acceptance criterion: a `diff` scope includes a package reached only through a `tier0` test file's import.
  Support log turn 4 updated to match.
- F2 (MEDIUM), fixed as proposed: D2's base requires the current tier fingerprint, empty args and the same module directory; D4's pruning keeps each tier's 20 newest whole-green commits; the D2 criterion adds the `-- -run X` and changed-command entries that are never a base.
- F3 (MEDIUM), fixed as proposed: D8 states the entered-row rule per path, routing, relaunch with and without a target (tackler and arbiter), the goto verb whoever calls it, and rename. The bound names the goto verb as the one bypass; the criterion and the auto-mode assumption follow; ledger Q21 updated.
- F4 (MEDIUM), fixed as proposed: D8 orders `pause_before` before gate evaluation and marks a row entered only after both; new acceptance criterion.
- F5 (MEDIUM), fixed as proposed, the retire branch kept with the trees' targets: new D14 retires `tree_scope_verify`, and the tree-scoped sites render `--gate <key> <targets>`.
  D3 lets explicit targets beside `--gate` replace the key's scope.
  D12's table gains the tree-scoped row; the `PATTERN-test-tiers` change line, Scope and the D3 acceptance criterion state the exception; new criterion and ledger Q22.
- F6 (MEDIUM), fixed as proposed: D4's bound names what a forged record lets through and accepts it explicitly in `internal/gatecli`'s doc, and states that inherited environment variables outside the tier's `env` are not fingerprinted.
- F7 (MEDIUM), fixed as proposed: D7 exempts runs whose args carry `-run`, `-skip`, `-list` or `-bench` and states the bound; the D7 criterion adds a filtered run that misses a package.
- F8 (MEDIUM), fixed as proposed: D13 replaces the discovered-count check with the first retagging card's `**Verify:**` line, run by `lyx gate card-verify`, which fails unless the hub's tier commands name their tags; the open risk names that line.
- F9 (LOW), fixed as proposed: D4 writes a whole entry when the carried part and the passed remainder together cover the tier.
- F10 (LOW), fixed as proposed, the keep branch: D3 says the darn form keeps its merge-base lint on every attempt.
- F11 (LOW), fixed as proposed, the refuse branch, moved from build to evaluation: D9's bare operand reads a bool or a number, and any other value is an evaluation error, so `open_window: "no"` stops the step as the Command producer's refusal does today; new acceptance criterion.
  Recipe build cannot refuse it, since the value is read at run time.
- F12 (LOW), fixed as proposed: the carry criterion now says any args other than the guard step's own `-run` filter break the carry unless they match the entry exactly.
- F13 (LOW), fixed as proposed: D10 says the inner run's hidden `--open-terminal` flag, which `--window` passes on, is what `Env.Setting` answers `batten.open_window` true for.
- F14 (NIT), fixed as proposed: the cost-against-gain sentence moved from D5 to support-log turn 4.

## Self-check

- fix-consistency LOW, ledger Q13 still says the operator's force: fixed as proposed.
- fix-consistency LOW, the reconcile Constraint omits the `tree_scope_verify` prune: fixed as proposed.
- fix-consistency LOW, the hub-change Constraint does not point at D13's Verify line: fixed as proposed.
- fix-consistency LOW, the Goal overstates what the key decides: fixed as proposed, the Goal names D14's exception.
- fix-claims MEDIUM and fix-way-forward MEDIUM, merged: bare `lyx config gate` opens an editor, so the Verify line could never pass: fixed as proposed, it reads `lyx config gate --resolved`.
- fix-way-forward LOW, a failed Verify line names no way forward: fixed as proposed, D13 names the Constraints' hub change and a rerun.
- fix-claims MEDIUM and fix-proposal MEDIUM, merged: a targeted relaunch whose target is already in effect writes the status unchanged: fixed as proposed, D8 and the criterion name the in-effect case.
- fix-claims MEDIUM, D4's "no standing deny covers it" is false, since shellguard refuses redirect writes: fixed as proposed, the bound says what the deny covers and what reaches the record.
- fix-proposal MEDIUM, after the deploy the tier-only loads miss old-tagged files on a branch not yet holding the retag: fixed differently, D5's loads list both spellings, D13's follow-up entry drops the old ones, and D13's bound sentence follows.
- fix-proposal MEDIUM, D7 exempts guard runs of a tier the key does not list: fixed differently, the guard step is no longer exempt, since each package of a guard group holds a test its filter names.
  The bound-only change was not made because it would leave that tier unchecked at the site.
- fix-proposal MEDIUM, `tree_scope_verify` doubles as tree scope's off switch: fixed differently, D14 makes a tree in `tree_scope` the on state and has reconcile empty `tree_scope` when `tree_scope_verify` was empty, so an off hub stays off; `ErrTreeScopeWithoutTrees` goes; the criterion follows.
- fix-proposal LOW, `all_targets` is a pattern, not a package set: fixed as proposed, D4 counts the packages it expands to.

## Disputed

None.

## Deferred

None.
