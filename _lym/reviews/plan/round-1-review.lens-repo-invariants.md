```yaml
lens: repo-invariants
outcome: reported
```

Repo-invariants lens: for each card, I checked test seams, doc text the card makes false, and card order. I found one blocking break, a missing guard entry in card 13, and four docs no card updates.

**Per card, invariant checked: held or broken**
- Card 1 — tier tag (TestBannedImports tier0): held. Card order (no earlier dependency): held.
- Card 2 — leaf tests NoStale with `lymdirs` (structural dirs import it): held. Geometry literals (no token in join, `+` or const context): held. `stringSlicesEqual` leaves with only the one other user the card rewrites (structuraldirs_test.go:90): held. Backgrounds `PATTERN-lym-files.md:36` and `PATTERN-fabric-boundary.md:58` and `docs/overview.md` are listed: held.
- Card 3 — TestEnforcement_BoltDirNameDeclaredOnce: **broken** (F1). geometryTokenOwners `-LYMHUB`, `_portals` and `_launchers` change in the same card: held. Comment at enforcement_test.go:229 ("fabricengine and runproc take the name from it") is now false: **broken, LOW** (F5). reedengine doc.go: **broken** (F3). burlerengine doc.go:127 names `fabricengine.StencilsDir`: covered only by the card's general comment rule, with no target listed (LOW, inside F3).
- Card 4 — `weftnameImportOwners` and the comment at TestEnforcement_FabricVocabulary:784 change in the same card: held. fabriccli doc.go:11 names `fabricengine.WarpBindingFileName`/`ShortnameFileName`: the general comment rule covers it, but no target lists it (LOW, inside F3).
- Card 5 — the engine mutation.go file comment ("only file that declares the Kind enum", mutation.go:3-4) is rewritten by the card itself: held.
- Card 6 — the result-type guard's `//lym:watch` gains fabriclayout: held. Tier tags: held.
- Cards 7, 8, 9 — no reflective seam test over the moved structs; doc.go edits listed: held.
- Card 10 — the reflective TestUnwireVerbResult_HasNoGitignoreField (unwire_test.go:223) only needs its qualifier retargeted: held. `PATTERN-pair-remote.md:98` listed: held. pairteardown doc.go:11 and :70 (`RemoveResult`, `LogArchiveDir`): covered only by the general rule, unlisted (LOW, inside F3).
- Card 11 — WorktreeEntry JSON tags unchanged: held.
- Card 12 — tier tags held. Refusal.Error with no allowlist entry: card 12 is fine (nothing imports fabriccore yet), but card 13 breaks it (F2).
- Card 13 — TestEnforcement_TestkitInvariant (only a production file importing a kit fails it; enforcement_test.go:54): held. Test-only-code guard: **broken** (F2).
- Card 14 — doc vocabulary: **broken** (F4). testOnlyCodeAllowed doc comment: **broken** (F6).
- Card 15 — tier1 tags held. Serial PATH test held. Allowlist entries land in the same card: held. testOnlyCodeAllowed doc comment: **broken** (F6).

**Findings**

- **F1 — BLOCKING, consistency, card 3 (`03-layout-hub-paths.md`, TestHubPaths).** The card wants literal paths such as `/h/_bolt/_lym/specs`. TestEnforcement_BoltDirNameDeclaredOnce walks every Go file, tests included (`scankit.All`), and fails any whole-token `_bolt` outside `internal/lymdirs/dirs.go`. So card 3 fails its own tier0 gate.
  - Evidence: internal/lymcwd/enforcement_test.go:526-575 (the pattern `(^|[^A-Za-z0-9_])_bolt($|...)`, `Filter: scankit.All`).
  - Fix: build the wants from `lymdirs.BoltDirName`, as card 2's TestClassifyPaths row already does.
- **F2 — MEDIUM, design, card 13 → card 14 (`cmd/lym#testOnlyCodeAllowed`).** From card 13, `internal/testkit/fabrickit` (a test-support package) imports `internal/fabriccore`. That puts fabriccore inside the test-only-code analysis, and no root reaches `fabriccore.Refusal.Error`. The allowlist entry only arrives in card 14, so the tier1 guard fails at any batch boundary between cards 13 and 14.
  - Evidence: pattern/PATTERN-test-only-code.md, the bound "a package that neither a main nor a test-support package imports is outside the analysis"; cmd/lym/testonlycode_linux_test.go:81 (tier1), :87-94.
  - Fix: move the `internal/fabriccore.Refusal.Error` entry into card 12 or card 13.
- **F3 — MEDIUM, consistency, card 3 (`internal/reedengine/doc.go:20`).** The doc says "The one fabricengine call, fabricengine.HubScratchDir in SpawnWatchdog". After card 3, spawnwatchdog.go:17/46 imports only `fabriclayout`, so the sentence is false beyond a qualifier swap. No card lists the file, and `internal/reedengine` is not in the decision record's Scope list.
  - The same unlisted-doc gap, at LOW, covers internal/burlerengine/doc.go:127 (card 3), internal/fabriccli/doc.go:11 (card 4) and internal/pairteardown/doc.go:11 and :70 (cards 10 and 3).
- **F4 — MEDIUM, consistency, cards 12/14 (`internal/fabricengine/livestate_doc_test.go`, "Naming: manifest, never snapshot").** That doc says "In fabric's own vocabulary, Snapshot names the `Snapshot: &lt;tag&gt;` trailer". Cards 12, 14 and 15 make "snapshot" the core's input (D11) in fabriccore and in the engine's doc.go executor section. No card updates or disambiguates the harness doc. The engine doc.go also already uses "snapshot tags" and `rec.Snapshot()` (doc.go:99-107, 248, 319), and card 14's doc section does not separate the two meanings.
- **F5 — LOW, consistency, card 3 (`internal/lymcwd/enforcement_test.go:229`).** The comment "fabricengine and runproc take the name from it" goes stale once the engine's `BoltDirName` moves to fabriclayout. Card 3 edits this test's comments only for HubPath and the reserved names.
- **F6 — MEDIUM, consistency, cards 14/15 (`cmd/lym/testonlycode_linux_test.go:41-46`, and `pattern/PATTERN-test-only-code.md` Rule).** The var's doc comment and the background both define an allowlist entry as "a reference or helper the tests of other packages share". The five loop and reader entries are neither, and neither card updates that text. This overlaps the handler's departure question; it is reported here only as doc falsification.

**Tool calls**
1. grep of pattern/, PATTERN.md, docs/overview.md and package doc.go files for moved names: the manifest did not cover doc text outside the engine (the first form came back empty).
2. The same search with a working pattern: the first command returned nothing.
3. reedengine/burlerengine fabricengine uses, leaf-enforcement test files, reflective tests over moved types: needed to judge doc falsification and seam tests.
4. unwire_test.go reflective test, livestate_doc_test.go, tests that read doc.go: needed to check the seam test and vocabulary docs.
5. The engine doc.go "snapshot" uses, the Kind-enum claim, the test-only guard's tier, and the TestkitInvariant import rule: needed for F2 and F4.
6. Other users of `stringSlicesEqual`, and `-weft` literals in the moved TestRecordsBranchName: needed for the card 2 and card 4 card-order checks.
