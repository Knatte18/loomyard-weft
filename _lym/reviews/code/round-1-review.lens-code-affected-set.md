```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I checked what reads it outside the touched lines.

**Per change, the set found, and whether it is covered:**

- **Four-value return → `*ending`; `stuckVerdict`, `gateStop`, `reportStuck` and the per-producer wrappers removed.**
  - Readers are all inside the package.
  - The one test caller, `prreview_wait_test.go:936`, is updated.
  - The repo-wide grep found no doc, stencil or rule file naming the removed symbols. The hits in `battenshed` (`reportStuck`) and `sheddriver` (`errorStop`, `handedOver`) are unrelated symbols of their own.
  - Covered.
- **`describeSessionTable` removed; the fixer, fix-review and Describe tables built through `soloSession.table`.**
  - `internal/shuttleparity/seats.go:126-128`: Holders now name `soloSession.table`. Covered.
  - `contracts/stencils/landing/landing-template-fix-review.md:4` says the value map is the one "landingshed's fixReviewTable builds". `fixReviewTable` still builds that map (`fixreview.go:113`). Covered.
  - `describe_test.go:311` asserts "describe model-spec", which `modelKey: "describe"` still produces. Covered.
- **Finalize's change-description error, "parse change description" → "read the change description".**
  - No test or doc asserts either text.
  - The commit message of `5064cc56` and `_lym/landing/summary.md` both name the change.
  - Covered.
- **"Earlier merge record unreadable" log text unified across both rows.**
  - No test asserts it. Covered.
- **`checkWiring` by reflection; message now "Deps.PullRequest.X is not set"; checking order follows field declaration order.**
  - No test asserts the wiring messages (grep for "is not set" under `internal/landingshed`'s tests found none).
  - The `PullRequestDeps` doc at `deps.go:400` names all four optional fields, matching `optionalPullRequestSeams`.
  - Covered.
- **Held read moved to once per pass; the land check's own halt on a failed held read removed; `syncParent`'s error path removed.**
  - `doc.go:201` and `doc.go:209-211` say Finalize reads `ParentHoldsTaskWork` first and a parent holding the work skips the owner checks. Still true.
  - `deps.go:140-147` says "read before the owner checks" and "a returned error halts the run". Still true.
  - `doc.go:223-224` on the land check names no held-read halt.
  - Covered.
- **`parentPuller` removed.**
  - No reader outside the package; `FastForwardPair` is still reached through `parentMerger` (`finalize.go:51`).
  - Covered.
- **Parent-Align reads `RowEntry` after the scratch move, and a nil `RowEntry` now reads as entry 0.**
  - `deps.go:382-384`'s doc of `RowEntry` makes no ordering claim.
  - `NewParentAlign` does not require `RowEntry`; before the change a nil value would have panicked.
  - No doc describes the old order. Covered.
- **Fixer and fix-review parsers share `parseVerdictReport`.**
  - `pattern/PATTERN-sole-parsers.md` names `ParseFixerReport` and `ParseFixReviewReport` as the sole parsers. Both remain the only entry points, and the shared reader is unexported inside the same package.
  - Covered.
- **Repository rules.**
  - The new functions are all reached from production (`PATTERN-test-only-code`).
  - The seat-literal scan's holder list is updated (`PATTERN-engine-parity`).
  - No PATTERN changes are declared for this round, so nothing is owed under "Declared PATTERN changes landed".
- **Commit messages and departures.**
  - The range has no `Pattern-Departure:` or `Loosened-Check:` trailers.
  - `fd8c599d` says "Verdicts, reasons and errors are unchanged"; the error-text change is in `5064cc56`, whose message names it.
  - Covered.

**Findings:** none. Every reader I found is either inside the diff or still consistent with it.

Outside this lens: `_lym/landing/summary.md` gives production comment lines as 2,568 → 2,570. That was true at `eb3feb72`, but HEAD has 2,573 after `811f9e31`. That is the handler's claims check, not this lens.

Tool calls:
1. Grepped the tests for assertions on changed messages, and non-test files for removed or renamed names. The manifest's grep covered removed symbols only, not message texts or stencils.
2. Grepped `doc.go`, `deps.go`, `loomcli`, `contracts` and `docs` for the held-read, wiring and row-entry wording. The manifest held no doc readers of these behaviours.
3. Read `doc.go:196-215`, `deps.go:138-148` and the land-check lines of `doc.go` to confirm the wording the second grep located.
