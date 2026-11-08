# Fixer report — discussion round 1

Target: `_lym/discussion/decision-record.md`, with the round logged in `_lym/discussion/support-log.md`.

## Changes

- F2 (fixed as proposed): D7 step 1 records that `cardChanges` and `impactset.ChangedPaths` return only paths.
  It reads base text through `fabricengine.CodeFileAt` (new), a warp read at a revision, since `AnchoredFileAt` reads the weft (`internal/fabricengine/trailercommit.go:151-153`) and `Fabric.MergeBase` works on the warp (`warpforward.go:45-54`).
  A worktree with no parent branch judges nothing and says so. New D7 criterion.
- F3 (fixed as proposed): D5 lists skipped packages apart from the count the `--max` cap left unbuilt, and dirty or locked skips do not count against `--max`.
  D6 retriggers only on capped leftovers, and the D6 criterion is rewritten.
- F4 (fixed as proposed): staleness reads only Go files tracked at HEAD; a modified or untracked Go file makes a package skipped, never stale. Both D5 criteria are rewritten.
- F5 (fixed differently): the review proposed discarding a package's map when HEAD or a hash differs after its tests.
  D5 discards only when the package's own file hashes differ, and the map records the HEAD read before the package as its commit.
  Discarding on any HEAD move would starve a slow package on a hub whose landings come faster than its tests (self-check, fix-proposal 4).
  A landing that leaves the package's files alone is dependency drift, which D5's age-limit bound already covers.
  New D5 criterion.
- F6 (fixed as proposed): each test in `lym gate testmap` gets `redundancy_test_timeout_sec`, a timed-out test is left out of the map, and D5's bound is restated in time. New D5 criterion.
- F7 (fixed as proposed): D7's bound now says a map test counts as a cover whenever its own function text is unchanged, so an edit in the diff to a helper or callee can produce a false contained verdict, bounded only by review.
- F8 (fixed as proposed): every error the step meets is a report line, never an exit code. A failed report write is logged at Warn. The D7 criterion is extended.
- F9 (fixed as proposed): the step is skipped when the existing report matches the current files by D8's match. The cost bound now counts the coverage builds, the `lym` build and the last test's timeout past the budget. New D7 criterion.
- F10 (fixed as proposed): the refresh child registers under `runproc.HubOwner` as `runproc.RoleTestMap` (new), spelled `test-map`. A live record means a refresh is in flight, so a restarted dobby starts no second one. D6 states the bound.
- F11 (fixed as proposed): D7's bound says a kept new test is listed as not judged with its keep reason, and that Code-Review's reading of it is the escape's only check.
- F12 (fixed as proposed): step 2 keeps the static drops only. The skipped, failed and no-blocks drops follow step 4's runs.
- F13 (fixed as proposed): D8 says a red report never matches and the marker names the red gate. The D8 criterion is extended.
- F14 (fixed differently): the review proposed naming files that differ from the map and are unchanged since the merge base.
  That hides drift in a file both the parent and the run changed (self-check, fix-proposal 5).
  Step 6 names the files whose text at the merge base differs from the map's hash, which is the map's drift from the run's base and never the run's own edits.
- F15 (fixed differently): the line moved from Auto-mode assumptions to Constraints, as proposed.
  The proposed citation said the merge runs `git merge --ff-only` in the parent's weft checkout.
  The code runs `f.code.MergeStart` in the parent's warp checkout under the weft write lock (`internal/fabricengine/merge.go:421-488`), so the record cites that instead.
- F16 (fixed as proposed): step 6 now reads "contained with its covering tests, largest overlap first".
- F17 (fixed as proposed): D8's rationale states the map-rebuild bound.
- Support log: round 1 entry, two rejected alternatives, and the Q5 and Q7 ledger entries updated.
- Gate repair: the gate's citation check found two cited symbols that do not exist yet without a `(new)` mark.
  Those were `runproc.RoleTestMap` in D6 and `fabricengine.CodeFileAt` in the D7 criterion.
  Both now carry `(new)` right after the closing backtick.

## Self-check

The self-check's own changes come from merged findings: proposal 2 and way-forward 1 are one finding, and consistency 2 joined them.

- fix-consistency, MEDIUM, a discarded package escapes `--max`: fixed as proposed. Only dirty or locked skips, made before any build, are outside `--max`.
- fix-consistency, LOW, D7's skip borrows D8's HEAD-based match: fixed as proposed. The skip takes D8's match over the current files in place of HEAD's tree.
- fix-consistency, LOW, the D6 criterion contradicts itself: fixed as proposed, with "when the prime HEAD is unchanged and".
- fix-consistency, LOW, the support log's ledger is stale: fixed as proposed (Q5, Q7).
- fix-way-forward, MEDIUM, an error report blocks its own retry: fixed as proposed. A report with an error line, a no-map package or a test past the budget never skips the step. Criterion updated.
- fix-way-forward, LOW, skips give no way out: fixed as proposed. D5 lists each skipped package with its cause, and D6's bound names the commit-or-remove way forward.
- fix-proposal, MEDIUM, a dobby stop does end the child: fixed as proposed after checking `internal/runproc/end.go:355-385`.
  D6 says a stop ends the child, and a dead dobby's child ends within D5's bound. Its slot and locks are OS locks released on exit (`internal/gateslot/doc.go:31`, `internal/lock/lock.go` on `gofrs/flock`). New D6 criterion.
- fix-proposal, MEDIUM, the step on a non-`go` scoper: fixed as proposed in the step itself. It judges nothing and says so when `scoper` is not `go`. New D7 criterion.
- fix-proposal, MEDIUM, error reports never redone: merged with fix-way-forward MEDIUM above.
- fix-proposal, LOW, any HEAD move discards a package: fixed as proposed (see F5).
- fix-proposal, LOW, F14 hides drift in a file both sides changed: fixed differently from the lens's bound-only proposal. Step 6 compares the merge-base text with the map (see F14).
- fix-claims, MEDIUM, `gates` entries are filled from the template at load: valid. It voids the reinterpretation the fix had added for F1.
  Fixed by removing the reinterpretation, its criterion, its Constraints line and its plan note, and stating the fill in D7. F1 is now disputed (below).
- fix-claims, LOW, the repeat cost leaves out the merge-base reads: fixed as proposed.

Every fork returned a report.

## Disputed

- F1, `premise false`. The review says a hub base file's tuned `gates` unit wins over the template whole, so `reports: [redundancy]` in the template never reaches this hub.
  Config load runs `yamlengine.FillMissing(spec.Template, stackDoc, spec.OpenMaps...)` (`internal/configengine/config.go:278`).
  `fillMapping` (`internal/yamlengine/fill.go:84-108`) appends every template key an existing mapping lacks and skips only `OpenMaps` paths.
  The gate module's open maps are `tiers`, `tier_parallel` and `build` (`internal/gateslot/config.go`), so `gates` is not one of them.
  `config.go:280-285` calls a name filled under a per-name map "the normal case".
  This hub's `webster: {tiers: [0, 1, 2], scope: all}` therefore loads with the template's `reports`.
  D7 now states this fact, citing the fill and this hub's `gate.yaml`, and names `reports: []` as the way to turn the step off on a key. A criterion pins both.

## Deferred

None.

## Rulings

None.
