# Fixer report — discussion round 2

Every finding's premise held against the code (`internal/gatecli/testmappackage.go:647`, `:675`, `:857-873`; `internal/gatecli/redundantmutants.go:541-543`; `internal/gatecli/testmaptree.go:28`, `:80`; `internal/gateslot/tiers.go:34-43`).
All changes are in `_lym/discussion/decision-record.md`.

## Changes

- F1 (fixed differently): D6's failed-listing sentence now says such a build is checked before its failure is written,
  for HEAD and for every modified or untracked file of the module, since no linked set is known;
  a failed check leaves the map unwritten and lists the package as unmeasured.
  The proposal checked only the files the package itself owns, which the self-check showed leaves the F1 premise open:
  a dirty linked package that makes the listing fail still writes a failed build under a clean HEAD.
  Added the D6 criterion, worded to cover an edit in the package's own files or a linked package's.
- F2: D3 now bounds each coverage build by `redundancy_build_timeout_min` and each test process by `redundancy_test_timeout_sec`,
  with what passing each bound means; D1's Bound now names the tiers' `timeout_min` and `test_timeout_min` among what map mode skips.
- F3: D2's first sentence names D7's failed-build exception; D8's Rationale names the mixed write a failed build can leave.
- F4: D4's bound on the tier's cap is conditioned on a host whose memory cap covers a process tree, with the one-at-a-time case beside it.
- F5: D6's Bound names the gitignored-file gap.
- F6: the D7 green-skip criterion states that no green record from before the map run exists.
- F7: D7's first sentence and the `PATTERN-gate-slot` constraint rank a tier asking for every slot above any count.

## Self-check

- fix-consistency, LOW, Review criterion lacks F7's every-slot clause: fixed as proposed, the criterion now carries it.
- fix-consistency, LOW, D8 body still says each run writes a whole map: fixed with own change, the sentence now says each writes its map at once.
- fix-consistency, LOW, D4 Rationale states the cap equivalence unconditionally: fixed as proposed, it now holds on a host whose cap covers a process tree.
- fix-consistency, LOW, D6's first check named only a build that compiled: fixed as proposed, the check now follows a build that compiled or failed.
- fix-proposal, MEDIUM, a dirty linked package that fails the listing passes F1's check (also noted by fix-way-forward outside its lens): fixed as proposed, the check reads every modified or untracked file of the module and the criterion covers a linked package's edit.
- fix-proposal, LOW, D8 Rationale's first clause contradicts its exception: fixed as proposed, the clause is limited to both builds succeeding.
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
