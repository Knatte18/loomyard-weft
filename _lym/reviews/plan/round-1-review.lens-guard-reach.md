```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: I checked whether each guard, refusal, branch or test row in cards 1–8 can be reached, and whether any guarded site recurs on a path no card guards.

**Per guard, reachable or not**
- Card 1, `StandingPassages` zero `Standing` for an empty Bolt dir, no reader, or a reader error: reachable.
- Card 1, `FileEntries` returning none for an unknown engine: reachable.
- Card 2, the `manifests` skip in `sweepOrphans` and `findRunByStrand`: reachable once a launch keeps a copy. The other root scans (activity, agentwait, sessionread, stepstop, carryover, enginesrun, attach) already skip a directory with no `run.json`, so the card's claim holds.
- Card 2, `copyManifest` with an absent record: reachable (a fake engine).
- Card 2, `copyManifest` warnings for an unreadable record or a failed write: reachable through the test rows.
- Card 2, "a start retry's failed attempt leaves no copy": the stated bound is false. The copy is made after `saveRunState`, and a later `awaitStartup` failure goes back through the retry loop with that copy still in place.
- Cards 3 and 4, a manifest write failure refuses the launch: this is a raw I/O error, which is exempt.
- Card 4, no warning for an `ErrHookUnrun` reason: reachable. `pluginhook.go:144` sets `Reason: err.Error()`, so the unwrapped text matches. It is reached only through the seams of `NewWithSeams`.
- Card 5, the nil-seam fallbacks: reachable, because production `New` passes zero seams.
- Card 6, `PrepareEach` errors (a `Prepare` error, an absent record) and the `seatsOf` builder-error path: reachable with fakes, but only card 8's `TestReport` reaches them.
- Card 6, the unmet-requirement seat skip in `TestSeats_CodexManifestHoldsClaudes`: not reachable. No `seatengine` seat sets `ForkSubagents` or `Requires`.
- Card 6, `Unmet` on producers: reachable (Merriam's `Run`, `loomcli#driverSpec`).
- Card 7, the scan rows (unlisted Spec literal, Spec literal with no `Prompt`, empty `Table{}`, method glyph, function-literal glyph): reachable.
- Card 7, the `internal/orchengine` skip: the state does not occur, because no Spec literal exists there.
- Card 7, the `seatengine.Table` literal scan as the guard for "every seat": a guarded site recurs unguarded. Seats are added outside Table literals.
- Card 8, the `LookupSeat` refusal: reachable through an unknown seat name.
- Card 8, the positional-argument refusal: reachable.
- Card 8, the unmet-requirement exit: not reachable for any real seat; only the test stub reaches it.
- Card 8, removing the temporary directory on every path: reachable.

**Findings**

1. **BLOCKING, design.** Cards 6 and 7, the seat catalog and the `seatengine.Table` literal scan.
   - The burler assist seat is built as a `&seatengine.Seat{` pointer literal (`internal/burlerengine/assist.go:115-124`).
   - `withAssistant` puts that seat into the review table and switches the chair to `assistedReviewerStencil` (`assist.go:152-160`); `review.go:146,157` call it.
   - `Profile.SeatTables` returns the bare `reviewTable` and `fixTable`, and the scan matches only Table literals, so it can never flag these seats.
   - So the assisted chair and the assistant seat, the case where Codex runs as an assist seat, escape both the catalog and the scan.

2. **MEDIUM, consistency.** Card 2's Intent says "a start retry's failed attempt leaves no copy", which is false.
   - `Runner.launch` copies the record right after `saveRunState`, then runs `awaitStartup` (`internal/shuttleengine/run.go:731-760`).
   - A failed startup returns through the retry loop (`run.go:605-615`) and leaves `<root>/manifests/<runID>.json` for an attempt that never ran.
   - No test row covers this, and the bound is stated wrongly.

3. **MEDIUM, design.** Card 6 adds refusal branches that no test of its own reaches.
   - The branches are `PrepareEach`'s `Prepare`-error and absent-record errors and `seatsOf`'s per-builder error.
   - `TestSeats_CodexManifestHoldsClaudes` fails on any builder error but has no failing-builder or failing-`Prepare` row.
   - Only card 8's `TestReport` reaches them (`08-shuttle-manifest-verb.md` lines 59-62).

4. **LOW, design.** The unmet-requirement seat skip in card 6's `TestSeats_CodexManifestHoldsClaudes`, and card 8's unmet exit for a real seat, guard a state that cannot occur.
   - `seatSpec` copies `seat.ForkSubagents` and `seat.Requires` (`internal/seatengine/prompt.go:230-231`), but no production seat sets either.
   - The only literals that set them are producers: `websterengine/runlevel.go:768-770`, `loomcli/driverspec.go:62-63` and `orchcli/start.go:224`.
   - The overview's Shared Decision gives Merriam's forks and the loom driver's background wake as its seat examples, but both are producers, not seats.

5. **LOW, consistency.** Card 7 names only Merriam (`internal/websterengine#Run`) as a producer that is logged and not compared.
   - `internal/loomcli#driverSpec` also sets `ForkSubagents: true` and requires `RequirementBackgroundWake` (`internal/loomcli/driverspec.go:62-63`).
   - So it takes the same unmet branch, and the test's expectation as written misses it.

6. **LOW, consistency.** Card 2's `copyManifest` warning gives one way forward ("remove whatever blocks `<root>/manifests`") for both its branches.
   - For the unreadable-record branch (a `manifest.json` that is not JSON) that step is wrong: nothing blocks the directory there.
   - The cause is the engine's record write (`02-manifest-record.md` lines 56-57, 77).

7. **NIT, design.** Card 7 skips `internal/orchengine`, which holds no `shuttleengine.Spec` literal.
   - The only orch literal is `internal/orchcli/start.go:217`, so this skip guards a state that does not occur. D1 licenses skipping D2's orch places, so the skip is harmless.

Outside my lens: card 1's `TestStandingPassages` rows (empty Bolt dir, no reader, reader error) repeat what `claudeengine/prepare_test.go:586-660` `TestPrepare_AppendsStandingPassages` already covers.

Tool calls:
- `claudeengine/pluginhook.go` and `codexengine/usersetup.go` reason handling: to confirm the `ErrHookUnrun` text match; the manifest lacked how `Reason` is formed.
- `rundir.go#createRunDir` and the `run.go` start retry loop: to check whether a failed attempt keeps a copy; the manifest lacked the retry path.
- `shuttleengine/requires.go` and codexengine's `RequirementSupporter`: to check how `Unmet` mirrors `Start`; not in the manifest.
- The other run-directory-root scans: to confirm they skip a directory with no `run.json`; the manifest listed them but not their skip logic.
- `ForkSubagents`/`Requires` setters across production: to check whether any seat reaches the unmet branch; not in the manifest.
