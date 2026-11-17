```yaml
lens: repo-invariants
outcome: reported
```

Repo-invariants lens: I checked each card's test seams and tier tags, the docs its change falsifies, and whether it can pass its gate after the earlier cards. Two invariants break: card 8 runs a spawning test at tier0, and card 10 can break card 6's test without listing it. Everything else holds.

**Per card, each invariant checked:**

- **Card 1**
  - Tier tags: held. `TestExpand` and `TestLeafInvariant_StandardLibraryOnly` are both tier0 and spawn nothing. The leaf test reads only its own package, so it needs no `//lym:guard`.
  - Doc text: held. The card extends `PATTERN-leaf-packages.md`.
  - Card order: held.
- **Card 2**
  - Seams: held. It adds a row to `targetTable`, and no `//lym:seam` marks a heddleengine struct.
  - Tier: held. It changes the existing tier0 `TestRender`.
  - Doc text: held.
    - The card rewrites the prompt-target paragraph of `heddleengine/doc.go` (doc.go:47-50).
    - It also changes the `Show` refusal text. Only `render_test.go` asserts that text, and the card changes `TestRender`.
    - `overview_test.go` checks the overview by substrings (overview_test.go:54,128), so adding to `overviewIntro` breaks nothing.
  - Card order: held.
- **Card 3**
  - Tiers: held. Its new subtests go into `TestWorktreeSeam_Events`, which sits in a tier1 file (worktreeseam_integration_test.go). The new `hubreconcile_test` test is tier1, like ensure_integration_test.go.
  - Doc text: held.
    - It lists `PATTERN-heddle-instructions.md:22-24`, `fabricengine/doc.go` and `hubreconcile/doc.go`.
    - `hubreconcile/doc.go:40` names only the HEDDLE.md render, and the card extends that file.
  - Card order: held.
- **Card 4**
  - Seams: held. `RenderRequest` gains `Root`, and no seam marker covers it.
  - Doc text: held. The card rewrites the codex-file sentence at `heddleengine/doc.go:13`. The new import record is a `.lym` cache under the heddle scratch directory, consistent with `PATTERN-lym-files`.
  - Card order: held. It uses card 1's `Expand`.
- **Card 5**
  - Doc text: held.
    - `docs/overview.md:711` ("the creel trust sentence in its prompt file") is listed.
    - The `codexengine/doc.go` text at :63, :86 and :126 is covered by the card's rewrite of `codexengine/doc.go`.
    - The `guards.go` file comment is listed.
  - Forced test edits: held.
    - Every test that reads `prompt.md` or compares the whole launch line is listed: `TestPrepare_Launch` (prepare_test.go:113-135), `SandboxMode` (:200-238), `ExpandsHomeInWritableRoots` (:269-279), `InstallsGuardsFromItsInputs` (:321-348) and `AppendsStandingPassages` (:382).
    - `TestPrepare_RefusesBeforeWriting` (:628) only checks that `prompt.md` is absent, and `TestPrepare_SkillsLoadFirst` matches by substring (skillload_test.go:302+).
- **Card 6**
  - Seams: held. The only seam marker on a shuttle struct is on `shuttleengine#Config` (for example envkit_test.go:52), and no card changes that struct. codexengine's `Config` and `launchLine` are unmarked.
  - Tiers: held. Both new tests are tier0.
  - Doc text: held. It extends `codexengine/doc.go`.
- **Card 7**
  - Tiers: held. The new tests are tier0 and read only temp files.
  - Doc text: held. It extends `shuttleengine/doc.go` and `claudeengine/doc.go`.
  - Card order: held. It uses card 1.
- **Card 8**
  - Tier tag: **broken** (finding F1).
  - Doc text: held for `claudeengine/doc.go`, which the card extends.
- **Card 9**
  - Tier: held. The new test goes in package `shuttleengines`, whose test file is tier0 (shuttleengines_test.go). Under `danger-full-access` no probe runs, no plugins are installed, and `readCheckout` uses `dotgit` alone (home.go:73-85), so nothing spawns.
  - Seams: held. `Codex` gains fields, and no seam marker covers it.
  - Doc text: held. It extends `PATTERN-shuttle-provider-seam.md` and `shuttleengines/doc.go`.
  - Card order: held. All callers of `NewFromConfig` (fixture_test.go:149 and shuttleengines.go:82) are on this card.
- **Card 10**
  - Tier: held. The new test is tier0. The row that skips as root names a prerequisite the host lacks, which is allowed.
  - Doc text: held. It extends `codexengine/doc.go`.
  - Card order: **broken** (finding F2).

**Findings**

- **F1 — BLOCKING, design.**
  - Location: card 8, `plan:internal/shuttleengine/claudeengine#TestRunPluginHook`.
  - Summary: the card plans `TestRunPluginHook` as tier0, but its rows run real `sh -c` processes through `RunPluginHook`, including `sleep 5; true` under a timeout.
  - Why it breaks: tier0 is "offline, no spawn", and subprocesses belong to tier1 (`pattern/PATTERN-test-speed.md:3,12-13`). The functions of one test file also name one tier (:24), so this test needs a tier1 file of its own. Its call sites stay outside the static `exec.Command` scan, so no scan would catch it.
  - Evidence: card `08-plugin-session-start-contexts.md:57,67-69`.
- **F2 — MEDIUM, consistency.**
  - Location: card 10's Edit list, together with card 6's `plan:internal/shuttleengine/codexengine#TestPrepare_SubagentTerms`.
  - Summary: card 10 adds `-c project_doc_max_bytes=&lt;n&gt;` to every Prepare launch line, but its Edit list names only "the four full-line Prepare tests".
  - Why it breaks: card 6 says `TestPrepare_SubagentTerms` builds each row's "want line" through `developerInstructions` and `tomlString`, the same full-line comparison the four tests use. Its want lines carry no `project_doc_max_bytes` setting, so card 10's codexengine tier0 card gate fails on a test card 10 does not list.
  - Card 5's rewritten `TestPrepare_AppendsStandingPassages` table may be in the same position under cards 6 and 10 if its rows compare whole lines. Card 5 does not pin that, so it is a risk rather than a confirmed break.
  - Evidence: `06-codex-subagent-terms.md:58-65,69`; `10-codex-instruction-files.md:10-18,41-42,66`.

**Tool calls**

1. Searched for `//lym:seam` markers and read the tier tags of the touched test files. My context had no seam-marker list and no tiers for those files.
2. Searched docs, `pattern/` and `PATTERN.md` for text the cards falsify. My context had not read the docs beyond the pattern files.
3. Read `heddleengine/doc.go`, `codexengine/doc.go`, `hubreconcile/doc.go` and the Codex lines of `docs/overview.md`. My context had not read these files.
4. Searched codexengine tests for `prompt.md` reads and full launch-line comparisons. My context lacked the full list of tests the cards force to change.
5. Searched for tests asserting the `Show` refusal text and `overviewIntro`. My context lacked the readers of these two strings.
6. Read how `overview_test` asserts the intro. My context lacked its assertion style.
