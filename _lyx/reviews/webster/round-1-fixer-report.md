# Fixer-report, webster round 1

## Changes

- F1, F2: restored the clauses on the `PATTERN-durable-vs-ephemeral-state` and `PATTERN-push-both-sides` index lines to their text at `0ffd168b`.
  The fork self-check showed the restore pushes `PATTERN.md` past the headroom line, so the two declared index lines (`PATTERN-hub-wide-config`, `PATTERN-config-layers`) were shortened and `PATTERN-recipe-setting-lookup` lost the words "executable-key" and "resolved" with its meaning unchanged.
  Fixed differently for the declared lines: they keep their declared content in fewer bytes.
- F3: `LoadConfig` names the local file for removal only when `holdsUnits` finds units in it (`internal/gateslot/config.go`).
  An empty or comment-only local file leaves the clause out; an unparsable one keeps it, since the load then names that file.
- F5: `TestLoadConfig` gained three rows: a refused local file under an anchor with `_lyx/` carries `; remove <local>, which takes no gate key` ahead of the role clause, and a local file under an anchor with no `_lyx/` and a comment-only local file carry no removal clause.
  The comment-only row also covers F3.
- F4: `TestWebsterGeometry` asserts that a machine `go_parallel: 0` fails with `load <hub file> and <machine file>`, and the `TestGateTest_Scenario` machine row holds `take the step it names, then re-run the same command`.
- F6: `TestSet` gained a `LayerMachine` row whose board dir has no `_lyx/` beside an anchor that has one; it wants the refusal and no file.

`lyx gate test --tags integration` over `gateslot`, `configengine`, `hubgeom` and `gatecli` passed; `go test ./internal/pattern` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: the `holdsUnits` doc comment omits that an unparsable file counts as holding units. Fixed as proposed: the comment now says so.
- fix-proposal, BLOCKING: restoring F1 and F2 verbatim grew `PATTERN.md` past the headroom line (15838 bytes against 15747 on main). Fixed with my own change, as described under Changes.

## Disputed

Nothing disputed.

## Deferred

Nothing deferred.
The headroom residue the gate reported (32 bytes over) is repaired: `PATTERN-hub-store-gc` now says "run no automatic gc or maintenance" and `PATTERN-hub-containment` drops "never into a worktree", both stated in their background files.
`PATTERN.md` is 15744 bytes against 15747 on main.
