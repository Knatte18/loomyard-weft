```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens. For each card I checked three things: reflective test seams and tier tags, doc text the card makes false, and whether the card can pass its own gate after the cards before it.

**Per card, each invariant checked, held or broken**

- **Card 1:** PATTERN.md lines and headroom: held. The background files it edits: held. Two other pattern files go stale (finding 5): broken.
- **Card 2:**
  - runproc import allowlist: broken. Card 2 adds the `lymcwd` import to runproc, but `allowedImports` in `internal/runproc/leaf_enforcement_test.go` (lines 15-19) is not on the card, so the leaf test fails.
  - Geometry owners row: held. Doc text in lymcwd, runproc and junctionnames: held.
- **Card 3:** fabric doc.go and comments: held.
- **Card 4:** test literals: held. `livestate_doc_test.go` is exempted on purpose by card 7.
- **Card 5:** docs: held.
- **Card 6:** test literals: held.
- **Card 7:**
  - Tier tag (tier0, the file's own): held.
  - The `//lym:guard` marker on its repository-scanning test: broken (finding 2).
  - Card order: held, since cards 2-6 clear every Go hit and the md hits are all on the card.
- **Card 8:**
  - New test file is tier1: held.
  - `TestConfigTemplate` and its doc: held.
  - `config.go`'s file header still describes two keys: broken (finding 6).
  - No reflective seam over `fabricengine.Config`: held.
- **Card 9:** no `//lym:seam` marker covers `PullResult` or `BranchOutcome`, so the new fields need no seam: held. Tier1 test: held. fabricengine already has a TestMain: held. doc.go extended: held.
- **Card 10:** new tier0 test file: held. The new test package has no TestMain calling `tmuxkit.Main`: broken (finding 1).
- **Card 11:** no doc or test change needed: held.
- **Card 12:**
  - fabricengine and boardengine doc.go, `boardkit.InstallReplay` doc, the newRoot comment: held.
  - `restoreReplayTip`'s "board replay" message: broken (finding 4).
  - New test tiers: held.
- **Card 13:** package doc.go: held. tier0 test: held. No TestMain with `tmuxkit.Main`: broken (finding 1).
- **Card 14:**
  - lymdirs doc extended: held. lymdirs has no test file pinning its constants.
  - Tier tags: held.
  - Its tier1 hubforge test needs `gitkit.HermeticGitEnv` in a TestMain: broken (finding 1).
- **Card 15:** tier tags: held. It relies on the TestMain from finding 1.
- **Card 16:** tier tags: held.
- **Card 17:** shuttleengine docs: held. `prepare_test.go` stays tier0: held.
- **Card 18:**
  - newRoot `Long` and the registration oracle: held.
  - New heddlecli test package (tier1, uses hubforge) has no TestMain with `tmuxkit.Main` or `HermeticGitEnv`: broken (finding 1).
- **Card 19:** reads only its own package directory, so no guard marker is needed: held. Tier0: held.

**Findings**

1. **BLOCKING · design · cards 10, 13/14, 18.** No card creates a `testmain_test.go` for the three new test packages: `internal/entrymerge`, `internal/heddleengine` and `internal/heddlecli`.
   - PATTERN-test-isolation requires every test package to run through `tmuxkit.Main`.
   - A package whose tests spawn git through hubforge also needs `gitkit.HermeticGitEnv`. That applies to heddleengine's tier1 tests (cards 14 and 15) and heddlecli (card 18).
   - The checks are tier0 and run at every card gate: `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` (`//lym:guard`, `cmd/lym/tmuxisolation_test.go:258-259`) and `cmd/lym/hermeticenv_test.go:148`. So cards 10, 13 and 18 fail their own gate.
   - Evidence: `pattern/PATTERN-test-isolation.md` ("Every test package runs its tests through `tmuxkit.Main`").
2. **BLOCKING · consistency · card 7, `plan:internal/lymcwd#TestEnforcement_BoltDirNameDeclaredOnce`.** The new test walks the whole repository from `scankit.Root`, but the card does not give it the `//lym:guard` marker that PATTERN-test-speed requires of a test reading files outside its own package. Its sibling scanners in the same file carry it (`internal/lymcwd/enforcement_test.go:202` and `:721`).
3. **BLOCKING · consistency · card 2, `internal/runproc#allowedImports`.** Card 2 gives runproc the `lymcwd` import, but `allowedImports` in `internal/runproc/leaf_enforcement_test.go:15-19` is not on the card. `TestLeafInvariant_AllowlistOnly` then fails card 2's gate.
4. **LOW · consistency · card 12, `internal/fabricengine#restoreReplayTip`.** The card lists it only under Uses. After the change it restores after any replayer's failure, yet its message still reads "restore %s after a failed board replay" (`internal/fabricengine/destroy.go:1985`).
5. **LOW · consistency · card 1 / D9.** No card updates these pattern files, which still describe Bolt-checkout paths through `BoardDir` or the `_board` literal:
   - `pattern/PATTERN-config-layers.md:31`: `<BoardDir>/.lym/config/<module>.machine.yaml`
   - `pattern/PATTERN-durable-vs-ephemeral-state.md:7`: "the sibling is `BoardDir(hub)`"
   - `pattern/PATTERN-cwd-resolution.md:42` and `pattern/PATTERN-fabric-git.md:25`: spell `_board`
   Under D9 these places come from `BoltDir`, and card 7's guard does not scan `pattern/`.
6. **LOW · consistency · card 8, `internal/fabricengine/config.go`.** The file header still says Config carries only BranchPrefix and Pathspec, "in one fabric.yaml file", and the card does not list the file. Card 8 updates only the `ConfigSpec` comment for the third key.

Tool calls:
1. Grepped the `//lym:seam` markers and reflective tests over the structs the cards change; the manifest did not cover seam markers.
2. Grepped docs and pattern files for the replay-seam names and `_board`/`BoardDir`; the manifest did not read pattern files beyond the listed ones or the docs that name the seam.
3. Read the replay-seam mentions in destroy.go, boardkit and fabric doc.go; I needed the exact lines.
4. Checked the `//lym:guard` markers in lymcwd and the TestMain requirement in PATTERN-test-isolation and its enforcement; neither was in the manifest.
5. Checked for lymdirs tests and the tier tags on the isolation guards, to tell whether the card gates run them.
