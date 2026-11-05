```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked the test seams, the doc text it falsifies and whether it can pass its own gate given the cards before it.

**Per card, each invariant: held or broken**

- **Card 1:** PATTERN.md overview lines match the declared texts: held. The two background files are listed: held. Card order: first, held.
- **Card 2:**
  - lymdirs file header, runproc `doc.go` role list and githubclient `doc.go` are all listed: held.
  - runproc and githubclient have leaf import guards (`leaf_enforcement_test.go`), and the card adds no import: held.
  - Test seams: none touched, held.
- **Card 3:**
  - creelengine `doc.go` is falsified: **broken**. `doc.go:225-227` says the strand-claim refusal "is the only refusal that carries a code" and that "a program claim from a non-orch pane … carr[ies] no code". The card puts `doc.go` under **Extends** and only adds a sentence.
  - `resolveSender`'s own doc says "no other refusal does" (`identity.go:46`). The card edits `resolveSender`, but its Intent does not say the comment changes: **broken**.
  - Test tiers: `TestResolveSender` tier0 and `TestCreelCLI` tier1 are existing files, held.
- **Card 4:**
  - New test files: `files_test.go` tier0 and `testmain_test.go` support, held. `TestSlackGeometry` goes in `hubgeom_test.go`, which is tier0 (`hubgeom_test.go:34`, 7 tier0 Requires, no spawn), held.
  - hubgeom `doc.go` teller list is extended: held.
  - The told-geometry "Bound packages" list in `pattern/PATTERN-cwd-resolution.md:32` does not gain `slackengine`, though card 4 states slackengine is told its Geometry under that entry: **broken (LOW)**. No test enforces the list.
- **Card 5:** configreg flag and key tests, the strictness set and PATTERN-config-layers' degrading list are all listed in the same card: held. `config_test.go` tier0: held. Order: needs card 4's `doc.go`, which comes earlier, held.
- **Card 6:** `api_test.go` tier0 over httptest, no spawn: held.
- **Card 7:** `inbound_test.go` tier0: held.
- **Card 8:** `outbound_test.go` tier0: held. Doc and seams: n/a.
- **Card 9:** `unboundedWaitFiles` gains `relay.go` in the same card that creates it: held. PATTERN-daemon-wakeups and PATTERN-child-processes run-owner bullet are listed: held. `relay_test.go` tier0: held.
- **Card 10:**
  - `excludedModules`, both seamsignature lists and `docs/overview.md` are listed: held.
  - `cmd/lym/registration_test.go` discovers modules (`TestRegistration_AllModulesRegistered`, map built by discovery), so no list changes: held.
  - `cli_test.go` tier0: held.
  - PATTERN-cli-cobra package naming (slackcli imports slackengine): held.
- **Card 11:**
  - Reflective seam test over `orchCLI` or `orchengine.Paths`: none exists (no `lym:seam` or `NilSeams` in orchcli or orchengine tests), held.
  - orchengine has no import guard, so the new slackengine import is held.
  - Doc text: orchcli `doc.go:12` lists the bring-up steps (dobby, runs strand, orch, watcher) without the relay step. orchengine `doc.go:35` calls the reload pointer "the one-line stencil", but it now carries Go-appended presence text. Both files are only under **Uses**: **broken (LOW)**.
- **Card 12:** beamcli `doc.go` is falsified: **broken**. `doc.go:209` says "`Deps.Tick` is the tick of the `hibernateengine.Carrier`…", and `:212`'s Bound names only the carrier. The card makes Tick run the carrier and then `EnsureRelayWhileAway`, yet lists `internal/beamcli/doc.go` only under Uses. The test tier (tier1 in the integration file) holds.
- **Card 13:** fabriccli `doc.go:103-110` describes the anchor move's end and restart sequence and its envelope (`dobby_start_error`) with no relay restart or `relay_start_error`. `doc.go` is only under Uses: **broken (LOW)**. No reflective test over `anchorDeps` or `anchorResult`: held.
- **Card 14:** no marker, no Go reader: held.

**Findings**

1. **MEDIUM, consistency, card 3 (`internal/creelengine/doc.go` under Extends).** The card makes a program claim from a non-orch strand pane a coded refusal. The package doc says the strand-claim refusal is the only coded one and the program-claim refusal carries no code. The card only appends a sentence, so it leaves the false text and should move `doc.go` under Edit to rewrite lines 225–227.
   Evidence: `internal/creelengine/doc.go:225-227`.
2. **LOW, consistency, card 3 (`internal/creelengine#resolveSender`).** The doc comment "The refusal of a strand claim … wraps ErrStrandClaimOutsidePane; no other refusal does" becomes false. The Intent should say the comment names both coded refusals.
   Evidence: `internal/creelengine/identity.go:46`.
3. **MEDIUM, consistency, card 12 (`internal/beamcli/doc.go`, only under Uses).** The package doc says the prime dobby's `Deps.Tick` is the hibernate carrier's tick, and its Bound speaks only of the carrier. Card 12 adds the relay ensure to that tick and should list `doc.go` (Extends, or Edit for line 209) with the new step.
   Evidence: `internal/beamcli/doc.go:209,212`.
4. **LOW, consistency, card 13 (`internal/fabriccli/doc.go`, only under Uses).** The verb doc describes what the anchor move ends and starts again, and its envelope fields. Card 13 adds a relay restart on every post-`EndDobbies` path and a `relay_start_error` field that the doc never names.
   Evidence: `internal/fabriccli/doc.go:103-110`.
5. **LOW, consistency, card 11 (`internal/orchcli/doc.go` and `internal/orchengine/doc.go`, only under Uses).**
   - orchcli's doc lists BeamUp's steps (dobby, runs strand, orch, watcher) with no relay step.
   - orchengine's doc says the reload pointer is "the one-line stencil that tells the session to read the role file", but `WithPresence` appends Go text to it.
   Evidence: `internal/orchcli/doc.go:12`, `internal/orchengine/doc.go:35,44`.
6. **LOW, consistency, card 4 (`pattern/PATTERN-cwd-resolution.md`).** Card 4 relies on PATTERN-cwd-resolution's told geometry for `slackengine`, but no card adds `slackengine` to that background's "Bound packages" list. The list then understates the bound set. Review alone holds it; no enforcing test was found.
   Evidence: `pattern/PATTERN-cwd-resolution.md:32`.

**Tool calls**

1. Grep for `lym:seam` markers on the changed structs, and the prerunseams test head. My context lacked any reflective seam marker search.
2. Grep for NilSeams/reflect in the affected packages' tests, and the runproc, orchcli, beamcli and orchengine doc lines. I needed those doc texts and the reflective tests.
3. Grep of the beamcli, fabriccli, orchcli, orchengine and creelengine `doc.go` lines on tick, anchor, bring-up, pointer and coded refusals. I needed the exact doc claims the cards change.
4. Grep for import-set guards, the registration oracle shape and test tiers. My context lacked the leaf guards and whether the registration test holds a list.
5. Grep of `hubgeom_test.go`'s tier and the PATTERN-dobby-runs text. I needed to confirm the tier of the file that card 4 extends.
6. Grep for an enforced told-geometry bound-package list. I needed to know whether a test pins PATTERN-cwd-resolution's list.
