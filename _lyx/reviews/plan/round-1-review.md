---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-conflict-session-skills.md:13
    summary: Card 4 adds landingshed.Deps.LanguageSkills, which TestLandingDeps_EveryFieldPopulated requires landingDeps to populate, so card 4's commit leaves internal/loomcli's suite red until card 5 changes landingDeps; card 4's own gate tests only its own packages and does not catch it.
    fix: Move the landingDeps change into card 4. Add to card 4's Edit list `internal/loomcli#landingDeps` with card 5's re-sign arrow (the `languageSkills []string` parameter after `parentName`), `internal/loomcli#loomCLI.loomPreRun`, `internal/loomcli#loomCLI.buildLoomShed`, `internal/loomcli#TestLandingDeps_EveryFieldPopulated`, `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`, `internal/loomcli#markDoneFixtureDir`, `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip` and `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`. In card 4's Intent, landingDeps gains the parameter and sets `landingshed.Deps.LanguageSkills` from it, loomPreRun and buildLoomShed pass nil for now, the four test callers pass nil, and TestLandingDeps_EveryFieldPopulated passes `[]string{"scribe:golang-testing"}`, asserts the field and drops the "sixteenth field" count. Remove the landingDeps line and those five test lines from card 5's Edit list, keep loomPreRun and buildLoomShed there, and reword card 5's Intent so wireLanding passes c.languageSkills where card 4 left nil. Fix card 4's ImpactSummary, which says every other literal is unaffected until card 5. lyx loom validate-plan --overlay reports no new finding for this target move.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/01-language-config-module.md:43
    summary: Card 1's Load marks every LoadOrTemplate error invalid and wraps it in ErrUnusable with the "lyx config language" way forward. That includes a present but unreadable language.yaml, which configengine leaves unmarked as a transient read error. Through card 6's new ErrUnusable case it becomes an invalid refusal that stops Master, with a wrong way forward, and it contradicts the content-only triggers of the refusal rows in cards 5 and 6.
    fix: In card 1's Load bullet, apply configengine.MarkInvalid, ErrUnusable and the "fix it with lyx config language from the prime" clause only to content failures (a LoadOrTemplate error for which errors.Is(err, configengine.ErrInvalid) holds, plus the loader's own skills checks). Return any other LoadOrTemplate error unwrapped, so webster's configLoadError default case and loom's existing transient handling classify it as transient. Add a TestLoad_InvalidFailsNamingKey row, or a row in a sibling case, with a directory where the board's language.yaml belongs, asserting that the error is neither configengine.ErrInvalid nor languageconfig.ErrUnusable. Reword card 1's ErrUnusable sentence so it marks a content failure.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-language-config-module.md:29
    summary: Card 1 adds the skills key that D7 calls a hatch, but its Intent never says what the key now lets through (a listed build skill sends code roles to raw commands that the session deny does not refuse when they are not module-wide or tier runs, outside the gate slots) or what bounds it (the template comment, the operator's per-hub choice, the Guard classification and the verify gate that still runs the hub's build).
    fix: In card 1's Intent, after the template.yaml paragraph, add one sentence stating what the skills key lets through and what bounds it, as D7's Bound says. A listed build skill sends every code role to its raw commands, such as goimports -w, golangci-lint run or a one-package go test, which run unrefused outside the gate slots. Bounding that are the template comment, the operator's per-hub choice, the Guard classification that keeps a pair from setting it, and the verify gates, which still run the hub's own build.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-burler-profile-key.md:16
    summary: Card 8 adds the language-skills key to the recipe profiles and to lyx burler run's profile YAML, so any recipe row or ad-hoc profile can opt a round's seats into the hub's language skills, but its Intent never states what the key lets through or what bounds it.
    fix: In card 8's Intent, add one sentence saying that the key lets a recipe row or a lyx burler run profile give both seats of a half the hub's language list after burlerSkills. It is bounded by D5, since it only adds skills to the seats' load turn and changes no rubric, lens, fan or gate, and by the list itself, a Guard key of the hub's language.yaml that a pair cannot set. An absent key keeps today's burlerSkills.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/01-language-config-module.md:54
    summary: The local-override row of TestLoad_InvalidFailsNamingKey names only configengine.LocalFile(anchor, "language"). With a told board dir, configengine drops the anchor's worktree and local layers when the anchor has no _lyx/ directory, so as specified the row never reaches the local-layer refusal.
    fix: In card 1's TestLoad_InvalidFailsNamingKey bullet, state that the local-override row creates the anchor's `_lyx/` directory (configengine.ConfigDir(anchor)) as well as the local file, and passes both Board and Anchor in the Dirs, so the local layer stacks and its refusal of `skills` is reached.
---

### [MEDIUM] Card 4 leaves loomcli's reflection test red until card 5

**Location:** `_lyx/plan/04-conflict-session-skills.md:13`

**Issue:** `TestLandingDeps_EveryFieldPopulated` (`internal/loomcli/landingdeps_test.go:41`) walks the `landingshed.Deps` that `landingDeps` returns by reflection and fails on any zero field.
Card 4 adds `landingshed.Deps.LanguageSkills`, but `landingDeps` gains its parameter only in card 5, so after card 4's commit that test fails.
Webster's card gate is `lyx gate test` over the card's own package directories plus the `//lyx:guard` tests (`internal/websterengine/doc.go:75`), and this test carries no guard mark.
Card 4's gate therefore passes, while its commit leaves `internal/loomcli` red, and card 4 is no longer independently committable.
Card 4's ImpactSummary claims every other `landingshed.Deps` literal is unaffected until card 5, which misses this test.
The proposed target move passed `lyx loom validate-plan --overlay` with no non-informational finding against a `{}` baseline that also had none.

### [MEDIUM] Card 1 classifies an unreadable language.yaml as invalid content

**Location:** `_lyx/plan/01-language-config-module.md:43`

**Issue:** `configengine` marks content failures `ErrInvalid` and leaves an unreadable file unmarked (`internal/configengine/config_test.go`'s `baseUnreadable` state expects `read config file` with no invalid mark).
`webster`'s `configLoadError` (`internal/webstercli/wiring.go:351`) relies on that split: an unmarked cause is transient (`re-run the verb`).
Card 1 re-marks every `LoadOrTemplate` failure invalid and wraps it in `ErrUnusable` with the `lyx config language` way forward.
Card 6 then routes any `ErrUnusable` cause to an invalid refusal carrying `config_invalid: true`, which stops Master.
A transient I/O failure would halt a run as a correctness failure and tell the operator to edit a file that has nothing wrong with it.
D3 licenses failing the load on a file that does not parse or on a bad entry, not on a read error, and the refusal rows cards 5 and 6 write list only content triggers.

### [MEDIUM] Card 1 states no bound for the skills hatch

**Location:** `_lyx/plan/01-language-config-module.md:29`

**Issue:** D7 names the `skills` key a hatch: a listed build skill sends agents to raw commands the session deny does not refuse when they are not module-wide or tier runs, outside the gate slots.
The rubric's attack-surface rule requires the card that introduces the key to state, in its Intent, what it lets through and what bounds it.
Card 1's Intent describes the template comment's wording, but neither what the key lets through nor what bounds it.

### [MEDIUM] Card 8 states no bound for the language-skills profile key

**Location:** `_lyx/plan/08-burler-profile-key.md:16`

**Issue:** Card 8 adds a new key to the recipe's Burler profile mappers and to `lyx burler run`'s profile YAML.
Card 7's Intent bounds the engine's `Profile.LanguageSkills` field, but card 8, which opens the key to recipe rows and to ad-hoc profiles, states no bound.
Its ImpactSummary says only that existing rows map to false.
Card 9 sets existing values and adds no key, so it needs no bound of its own.

### [LOW] Card 1's local-override test row does not reach the local layer

**Location:** `_lyx/plan/01-language-config-module.md:54`

**Issue:** `configengine`'s `load` (`internal/configengine/config.go:157`) clears `Dirs.Anchor` when a board dir is told and the anchor has no `_lyx/`, so no worktree or local layer stacks.
As specified, the row seeds only `configengine.LocalFile(anchor, "language")` under `.lyx/config/`, so without an anchor `_lyx/` the local file is never read and the expected refusal never fires.

## Rejected

- Card order, focus item 1: every card that consumes `AppendSkills` (cards 2, 3, 4, 7), `Load` (cards 5, 6, 7) or `Profile.LanguageSkills` (card 8, after card 7) comes after the card that creates it, and card 9 follows card 8. Card 7's edit to `loomCLI.wire` reads `c.languageSkills`, which card 5 adds. The only dependency on a later card is F1's red test.
- Member glyphs and re-sign arrows, focus item 2: every Edit card names member glyphs for the symbols its Intent changes. `burlerengine#New`, `Profile.reviewTable`, `Profile.fixTable` and `loomcli#landingDeps` carry arrows matching their new parameters. Every other member keeps its signature: `DarnTable`, `Run`, `recoverSpawn`, `buildConflictSpec`, `NewMergeIn`, `NewFinalize`, `wire`, `wireHub`, `configLoadError`, `decodeProfile` and the profile mappers.
- Fidelity, focus item 3: D1, D2, D3 (including the empty template default, the no-board degrade and the refusal rows), D4 (load once per process in loom's `wire`, webster's `wireHub` and burler's `wireHub`, `recover-batch` loading in its own process), D5, D6 (`scribe:testing` in card 2), D7 and every Constraint are carried. The alphabetical `landing`/`logger` slot and `TestModules_EveryTemplateKeyClassified` are both handled in card 1. Every production deps literal (`internal/loomcli`, `internal/webstercli`, `internal/landingshed`) and every `burlerengine.New` caller is covered. Card 5's `wireLanding` extraction is the means of telling the list through one tested site rather than new scope.
- Test economy, focus item 4: no existing test drives Merge-In into a conflict session (`publish_integration_test.go` stages none), so card 4's new integration test is justified. The loom recipe tests keep one invariant per file, so card 9's new file follows the package's convention. The other new tests extend existing tables. The `internal/landingshed` budget row sits at its count, 86, and card 4 raises it; `loomcli`, `webstercli` and `loomrecipe` have headroom for their new tests.
- Refusal reachability, focus item 4: the new loom row is reached by `TestWire_LanguageSkills`' empty-entry row, the webster case by `TestWireHub_LanguageSkills`' leading-space row, and burler's by its `HubWithInvalidLanguageList` subtest. The non-boolean `language-skills` refusal is reached by `TestBurlerHalfEntries_ColorSegment`'s new subtests and by `TestDecodeProfile`. F5 is the one unreachable row.
- Verify coverage, focus item 6: `go test ./...` and `go test -tags integration ./...` cover every targeted package, `llm` and `tmux` are compiled by `go vet`, and the order matches the plan template.
