---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-shell-guard-table.md:87
    summary: Card 2 moves the shell constants with their comments but says nothing about the per-row guardrail comments on today's standingDenies entries, which D5 requires moved unchanged and which the standingDenies rewrite would otherwise drop
    fix: In card 2's Intent, after the sentence at line 87 that starts The Move group relocates, add one sentence saying that each shell row's comment on its standingDenies entry in settings.go moves unchanged onto that row's entry in Rows in internal/shellguard/rows.go. That covers python's A guardrail, not a barrier block (what the pattern lets through and falsely denies), the raw-go and in-place-edit Installed for every run lines with their guardrail sentence, computed-rm's paragraph (the dialog it pre-empts, what it falsely denies and lets through) and machine-sleep's lines. Only the Ending with `; true` guarantees exit 0 sentence, which describes Claude's command shape, goes onto renderShellGuard. No target changes.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-shell-guard-table.md:102
    summary: TestCheckTexts plants a line break in a steer and expects checkTexts to refuse it, but card 2 defines the steer set as quote, double quote and backslash only (today's steerTextForbiddenChars), so that test row fails as specified
    fix: In card 2's Intent at line 102, change the TestCheckTexts sentence so the planted steer cases are a single quote, a double quote and a backslash, and the planted notice cases are those three plus a line break, each refused naming the row. This matches the character sets the card carries over from claudeengine at line 84 (D5's the checks that sit in claudeengine today). Leave forbiddenSteerChars and forbiddenNoticeChars as the card defines them.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/06-engine-loads-claude-config.md:97
    summary: The way forward loadConfigOrRefuse prints, lyx config claude --set or lyx config claude, cannot clear the card's own named input, a binary key in claude.local.yaml, since --set cannot remove a key and the bare verb edits the module's own file, not the local layer
    fix: "In card 6's Intent at line 97, change the wrapped text to claudeengine: %w; way forward: correct or remove the key in the file the error names, with lyx config claude --layer <default|worktree|local> (or --set for a value). Extend the three engineWithInvalidConfig rows in TestPrepare_RejectedBeforeArtifacts, TestOpenLaunch and TestPromptCacheLifetime to also assert the refusal names --layer. The form stays within D2's way forward (lyx config claude), and configcli already takes --layer."
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/05-claude-config-module.md:31
    summary: Card 5 lists internal/configreg#checkLoad under Uses, but its CheckLoad is built inside shuttleengines over claudeengine.LoadConfig and configreg reads ConfigModule.CheckLoad as is, so no code of this card uses the unexported configreg generic
    fix: Delete the Uses section of card 5 (the Uses header line and its one bullet internal/configreg#checkLoad). Checked with lyx loom validate-plan --overlay; it reports no blocking finding.
---

### [MEDIUM] Card 2 drops the per-row guardrail comments D5 says move unchanged

**Location:** `_lyx/plan/02-shell-guard-table.md:87`

**Issue:** D5 says the shell rows move "with their patterns, steers, notices and comments unchanged".
In today's `internal/shuttleengine/claudeengine/settings.go` (`standingDenies`, lines 268–318), the comments that say what each guard lets through and what it falsely denies sit on the `standingDeny` entries, not on the constants.
Examples are the python row's "A guardrail, not a barrier. The pattern lets through python as an argument, behind bash -c …" and the computed-rm row's paragraph on the Claude Code dialog it pre-empts.
Card 2's Move group carries only the constants' comments to `rows.go`, and the card rewrites `standingDenies` as the Claude rows plus a `renderShellGuard` loop.
Following the card as written deletes these entry comments: the only record of each guard's known false positives and gaps would vanish when the table moves to its new home.

### [MEDIUM] TestCheckTexts expects a refusal that the specified steer check never makes

**Location:** `_lyx/plan/02-shell-guard-table.md:102`

**Issue:** Line 84 defines `checkTexts` as refusing a steer holding a single quote, a double quote or a backslash, and a notice holding those or a line break.
That carries over today's `steerTextForbiddenChars` (`'"\`) and `noticeTextForbiddenChars` (that plus `\n\r`) from `settings.go:418–421`.
Line 102 has `TestCheckTexts` plant "a quote, a backslash and a line break in its steer and in its notice" and assert `checkTexts` "refuses each".
A row whose only defect is a line break in its steer passes the line-84 check, so the test row fails as written.
The implementer would have to pick a side with nothing to choose by.
Keeping the carried-over sets and planting the line break only in the notice follows D5's "the checks that sit in `claudeengine` today", and changes no shipped behavior.

### [LOW] The invalid-config way forward cannot reach a local-layer key

**Location:** `_lyx/plan/06-engine-loads-claude-config.md:97`

**Issue:** The refusal's named input, and the input of all three `engineWithInvalidConfig` test rows, is a `claude.local.yaml` holding `binary: /x`.
`configengine.Layers` refuses it through `ValidateLocal`.
The way forward it prints is `lyx config claude --set` or `lyx config claude`.
`--set` only writes a value: it cannot remove the key, and a set on the local layer is refused for the same Executable category.
The bare `lyx config claude` edits the module's own file (`configcli.go:553`: "default: the module's own file"), not the local override.
The operator has to read the file path out of `configengine`'s text and know about `--layer local`.
Naming `--layer` keeps the advice inside D2's "`lyx config claude`" and makes it reach every layer the error can name.

### [NIT] Card 5 lists configreg's unexported checkLoad under Uses

**Location:** `_lyx/plan/05-claude-config-module.md:31`

**Issue:** `checkLoad` (`internal/configreg/configreg.go:89`) is an unexported generic that adapts a typed loader.
Card 5 builds the `claude` entry's `CheckLoad func(configengine.Dirs) error` inside `shuttleengines`, which cannot call it.
`configreg.Modules` copies `ConfigModule.CheckLoad` straight into `Module.CheckLoad`, so nothing in the card uses `checkLoad`.
The entry is no other card's target, so it adds no dependency edge; it only misdescribes the card's reads.

## Rejected

- Fidelity: every decision D1–D10 and every Constraint has a carrying card: D1 and D2 in cards 5 and 6, D3 in card 7, D4 across cards 4, 7 and 8, D5 and D6 in card 2, D7 in card 3, D8 in card 9, D9 in card 10, and D10 split across cards 2, 3, 5, 7, 9, 10 and 11.
  `creelcli` does not exist in this tree, so the creel-core constraint asks for no adaptation here.
  Card 7's extra `renameDenyPython` branch, for a document holding both keys, is an edge D3's rename needs and reports rather than prunes; it is not unlicensed scope.
- Card order: no card uses something a later card introduces.
  `entry` (card 5) comes before its edits in cards 6 and 9 and its use in card 10.
  `retiredCovering` (card 4) comes before card 7, `shellGuardInstalled` and the golden test before their later edits, and `Rows` (card 2) before card 3.
  Between cards 7 and 8 reconcile would prune the retired keys, but that intermediate state never deploys.
- Call-site completeness: every current caller of `shuttleengines.New` (15 sites) and of `reedengine.New` is listed on card 6 or card 9, including the `reedengine` package's own test constructors.
  The only `claudeengine.New()` users that start real runs are the six llm fixtures card 6 names.
  The others parse, audit or render hook context only.
  Adding the `shuttleengines` import to `hubgeom`, `pairteardown` and `configreg` forms no cycle: none of the three is in `go list -deps ./internal/shuttleengines`.
- `New()` with zero `Dirs` does refuse in the config-reading methods, as card 6 claims: `configengine.load` errors with "neither an anchor nor a board dir was told" before any degrade.
- Test economy (focus item 3): the 64-entry golden pins byte identity, which no existing test does.
  `shellguard`'s three tests cover the import allowlist, the text check and pattern semantics, which the golden cannot see.
  `TestStandingDenies_RenderEveryShellGuardRow` catches a row added to the table but left unrendered, which the golden also cannot catch.
  Card 3 adds rows to the existing self-test rather than a new test.
  No per-symbol test is planned.
  Budget arithmetic checks out against `test-budget.yaml` and today's counts: `claudeengine` peaks at 110 against its raised 113, `shuttleengines` reaches 5, `configsync` 9, and `shellguard` holds 3.
- Member glyphs and re-sign arrows (focus item 4): every non-test Go Edit target whose Intent names symbols lists member glyphs.
  Every changed signature carries its arrow: `NewFromConfig`, `shuttleengines.New` and `build`, `buildSettings`, `buildDenyNotice`, `claudeBinary`, `resolvePromptCacheTTL`, `reedengine.New`, `WatchNames`, `watchNames` and `repairNames`.
  The remaining edited members keep their signatures.
- Attack surface and reachability (focus item 5): card 4 states its bound (admits only what the template key admitted, with no shape check on a retired unit) and card 8 states its (one layer, two registered modules, no value change).
  Each prescribed refusal has a named input and a reaching test: the retired-key refusals in `TestValidateLocal`, the move's `FileError` in `TestReconcile_FailureNamesItsFile`, the empty-scrub panic in `TestEngine_SocketAndSessionName`, and the invalid-config refusal in three test rows plus `TestNew`.
  Card 2's unknown-switch panic is a fail-loud wiring guard and needs no per-row test.
  `shuttleengine.LoadConfig` decodes without `KnownFields`, so a stale `claude_deny_python` at a base or worktree layer loads, as the open risk and card 7's test row expect.
- `## verify:` (focus item 6): it runs build, the three `vet` tags, the untagged tier and the integration tier over `./...`, in the stencil's order, which covers `shellguard`, `claudeengine`, `shuttleengines`, reed and the watchdog packages.
  The tmux-tagged reed tests are compiled, not run, which the rubric exempts.
- Cards 10 and 11 (focus item 7): every Edit and Delete card's ImpactSummary states a blast-radius conclusion rather than restating its Intent.
  Card 11 is a Prosa card over package doc comments that cards 5 and 8 cannot target beside their members, so it is an independent unit.
