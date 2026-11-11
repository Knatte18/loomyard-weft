# Fixer report — discussion round 1

## Changes

Files changed: `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

- F1 (BLOCKING), fixed as proposed: D4 compares every covered package from the map's own `Commit` to the working tree,
  reading content at that `Commit` through `fabricengine.Fabric.CodeFileAt`.
  D5 drops the tier-base hash check and widens when the `Commit` is unreadable in the object store.
  D4 states that a change on main since the build counts as a change.
  The D4 and D5 criteria are rewritten, and the hit-rate Open risk is gone.
  The fixer also made its own change: D2's covered-package field now records every file the directory tracks, non-Go included,
  so that D5's non-Go rule still has a record to compare against once the tier-base diff is gone.
  The support log's Turn 2, Q5 and Rejected alternatives record the move.
- F2: D5 adds a miss for the importing package's own Go files, test files included, that differ from the map's `Files`.
  Fixed differently: the proposed comparison "as well as every covered package's field" was not added,
  because F1 removes the covered-package hash check, and D4 now diffs covered packages instead.
  The D5 criterion names an edit to one of B's own test files.
- F3: D3 has a Bound naming the spawn scan's blind spots, and D2's third bullet cites `testcover.ScanSpawns`.
- F4: D2 skips a package when any covered package's directory is dirty,
  and checks every covered package's files for changes made while the tests ran.
  A new D2 criterion covers the dirty case.
- F5: the marker holds the build's `Commit` and `BuiltAt`; a mismatch is a D5 miss.
  The PATTERN-tolerant-records constraint and the D5 criterion name it.
- F6: the PATTERN-fabric-boundary constraint names `fabricengine.Fabric.CodeFileAt`.
- F7: the Open risks `gate.yaml` question is dropped, Scope Out names the operator's `lym config gate` step, and Q8 is an auto-pick.
- F8: the `testmap-upkeep` question is reworded as proposed.
- F9: the first Auto-mode assumption and the tier-2 Open risk are replaced by a measured cost-split Constraint.
- F10: D5 counts an added or removed named import as a miss. The self-check then narrowed the rule, see below.
- F11: D4 pairs same-named functions as one group; D5 widens when a file holds more `init` functions than at the map's `Commit`.
  A D4 criterion covers it.
- F12: the D3 criterion allows D6's over-budget whole run.
- F13: D7 and its criterion add `whole: lym binary` and `whole: over budget`.
- F14: fixed differently: the first D5 criterion now names a Go file holding an `init` function added since the map's `Commit`.
  The proposed wording, "a Go file added between the map's build and the comparison point", was not used.
  Under F1 such a file is a D4 diff and misses only through D5's own rules.
- F15: the Notes bullet says the kept set is built from the map's `Blocks` keys.
- F16: the bullet is cut down to its task-specific part, the D2 fixture module's hermetic git.

## Self-check

Changes made after the self-check:

- D2's dirty check covers any modified or untracked file.
- The D2 dirty criterion says B's map is left as it was.
- D4's rationale states the non-body bound.
- A new Open risk names non-body landings on main.
- D5 exempts an added standard-library or covered-package import, and its Bound names the standard-library `init` let-through.
- The D5 import criterion is updated to match.
- The support log updates Q6, the Turn 3 board-scope 2 line and the round 1 entry.

Fork findings:

- fix-consistency, LOW: Q6 contradicts D5's import and `init` rules. Fixed as proposed.
- fix-consistency, LOW: the Turn 3 board-scope 2 line points to a removed Open risk. Fixed as proposed.
- fix-consistency, LOW: "writes no map" in the D2 dirty criterion disagrees with today's skip. Fixed as proposed.
- fix-consistency, LOW: D2's dirty check covers Go files only, while the covered field records non-Go files. Fixed as proposed.
- fix-claims: no findings.
- fix-proposal, MEDIUM: diffing from `Commit` removes misses only for body changes, and non-body landings still miss across the build.
  Fixed as proposed: the D4 rationale is reworded and an Open risk added.
- fix-proposal, MEDIUM: F10's import rule widens on an added `fmt` import. Fixed with the fork's narrower rule, adapted:
  the exemption covers an added covered-package or standard-library import only.
  A removed import still counts, because the map cannot tell whether the package stays linked.
- fix-proposal, LOW: a dirty, widely imported package in the prime blocks most map builds. Disputed, see below.
- fix-way-forward: no findings.

## Disputed

- Fork finding fix-proposal 3 (F4's skip blocks most builds while the prime is dirty), ground `every fix worse`.
  The only alternative named, recording covered packages' hashes from the files the build read, breaks D4.
  D4 reads a changed file's content at the map's `Commit` through `fabricengine.Fabric.CodeFileAt`, and diffs it to find changed functions.
  Blocks built from dirty files would then be diffed against `Commit` content the build never saw,
  so a change could match no recorded function and select nothing.
  The skip leaves the old map in place, and D4 still selects correctly from its older `Commit`,
  so the cost is more D5 misses and never a let-through.

## Deferred

None.

## Rulings

None.
