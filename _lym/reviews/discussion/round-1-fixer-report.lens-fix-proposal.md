```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I tested each proposal the fixer carried out against a concrete case or a sibling it could break.

Proposals I tested and found sound:
- **F1:** `redundancy_test_memory_mb` must be at least 1 (`internal/gateslot/config.go:295-300`), so the divisor is never 0.
- **F2:** outside `_test.go` files, `lymbin` is imported only by `internal/testcover/spawnscan.go` and `internal/impactset/graph.go`. No hidden non-testkit helper reaches `lymbin`, so dropping the build loses no test.
- **F3, F5 (the refusals), F6 (the package set), F7, F8, F10, F11:** I could not build a failing case for any of them.

Findings:

1. **MEDIUM. D6's new Bound misses the window before the first check.** Concerns the D6 Bound, "an edit made after one check and reverted before the next goes unseen", paired with D6 now placing its first check "once a package's coverage build has listed the linked packages and compiled".
   - Concrete case: package B's build runs one package ahead, during A's tests (D2). A file in B is edited while B compiles and reverted before B's first check. Both of B's checks pass, and the map holds coverage of the edited code under HEAD's hashes.
   - That edit came before B's first check, not "after one check", and nothing checks at the start. The stated Bound therefore understates what goes unseen.
   - The Bound needs to read "an edit made before a package's first check, or between its two checks, and reverted before the next check goes unseen".
   - Evidence: the review's own F4 premise (`testmappackage.go:187-188`, `:224-236`): the build starts ahead, and its list and compile run in one goroutine with no check before them.

2. **MEDIUM. D1's new Bound contradicts D7.** Concerns the D1 Bound, "What bounds that is that the run is on command only and writes nothing but maps."
   - D7 says "It writes a log, and no run record…". A map run writes a log as well as maps.
   - Rewording the line to "writes nothing but maps and its log" fixes it.

3. **LOW. D6's checks have no place for a package that is never built.** Concerns the D1 line from F6, "A package with no mapped test gets a map of its file hashes with no tests, as today", paired with D6's "Once a package's coverage build has listed the linked packages and compiled…".
   - That package has no build (`testmappackage.go:131-132` writes it before any build or dirty check), so D6's first check never happens. The record does not say whether the HEAD and dirty checks apply before that write.
   - Concrete case: a dirty, untracked file in a package with no tests reaches the map's file hashes under HEAD's stamp.

4. **LOW. D7's failed build versus D6's new check order.** Concerns D7, "it still writes the map of every package D6 lets through" (from F9), paired with D6's check order and D7's "A failed build records the failure on the package's map".
   - A build whose linked-package listing fails never reaches D6's first check, which sits after listing and compiling (today `build.linked == nil` writes the failure directly, `testmappackage.go:161-166`).
   - So "every package D6 lets through" leaves it open whether a failed build's map is written.

Tool calls:
- grep of `redundancy_test_memory_mb` validation and of `lymbin` importers outside tests: my context lacked F1's divisor floor and F2's reach.
- `sed` of `internal/gateslot/config.go:285-300`: my context lacked the bound check's minimum value.
