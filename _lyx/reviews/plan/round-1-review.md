---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-webster-declared-changes.md:12
    summary: Cards 9 and 13 add parameters to the webster test helpers renderRecoveryWithUncommitted, renderMaster and renderForkWithPack, but list those member glyphs without the re-sign arrow
    fix: "In card 9's Edit group, replace the bare `internal/websterengine_test#renderRecoveryWithUncommitted` line with `internal/websterengine_test#renderRecoveryWithUncommitted` -> `func renderRecoveryWithUncommitted(batch batcher.Batch, repoRoot, anchorRoot, stencilsDir, specsDir, failureDigest, uncommittedPaths, notePath, parent string, verifyTiers, patternChanges []string) (string, error)`, and the bare `internal/websterengine_test#renderMaster` line with `internal/websterengine_test#renderMaster` -> `func renderMaster(batches []batcher.Batch, anchorRoot, repoRoot, stencilsDir, notePath, parent string, patternChanges []string) (string, error)`. In card 13's Edit group, replace the bare `internal/websterengine_test#renderForkWithPack` line with `internal/websterengine_test#renderForkWithPack` -> `func renderForkWithPack(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, layout websterengine.PackLayout, pack []planindex.CardPack, verifyTiers, renameTools []string, planVerify string) (prompt, packFile string, err error)`, and the bare `internal/websterengine_test#renderRecoveryWithUncommitted` line with `internal/websterengine_test#renderRecoveryWithUncommitted` -> `func renderRecoveryWithUncommitted(batch batcher.Batch, repoRoot, anchorRoot, stencilsDir, specsDir, failureDigest, uncommittedPaths, notePath, parent string, verifyTiers, renameTools, patternChanges []string, planVerify string) (string, error)`. Each arrow is the helper's current signature in internal/websterengine/template_test.go plus the parameters the card's own Intent adds."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-pattern-directive-changes.md:38
    summary: No card states D5.4's bound on the declared-changes route, which lets every reviewer, judge and implementer judge a listed entry by declared text instead of PATTERN.md's current text
    fix: "In card 8's Intent, after the paragraph that opens with D5.4's render half, add the bound as one sentence: a declaration lets every reviewer, judge and implementer follow and judge a listed entry by its declared text, never PATTERN.md's current text; it covers only the entries the decision record's `## PATTERN changes` lists, only through bullets of the shape `loom-discussion-spec.md` defines (card 5), appended after Discussion only through `lyx loom decision add`'s `--by parent|operator|arbiter` caller guards (card 7), and the Webster-Review rubric checks that the declared text lands (card 17). Add a matching clause to the `pattern-fragment-changes.md` bullet, so that the fragment's rule names that only the entries listed there are judged by their listed text."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-review-loom-declared-changes.md:43
    summary: Card 10's refusal-spec row calls the Bouncer judge call's malformed-record failure a correctness halt, but degrade returns Stuck, which the perch's on_stuck routes to the Burler-round row, so the run halts there with the Burler's message
    fix: "In card 10's refusal-spec bullet, describe the Bouncer row as what the code does: the Bouncer's judge call degrades to Stuck with no judge spawned, the perch's `on_stuck` (for example `Plan-Bouncer` to `Plan-Burler-Review` in `contracts/recipes/loom-recipe.yaml`) routes it to the Burler-round row, and that row's `Engine.Review` refuses on the same record and halts the run, so the operator sees the Burler-round row's message. Keep the Burler-round row as the correctness halt with the stated way forward. Either fold the Bouncer case into that row's trigger, or keep a separate Bouncer row whose outcome cell says the Stuck is routed to the Burler round and is not itself a halt."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/11-plan-gate-context-checks.md:58
    summary: Card 11 makes validate-plan load language.yaml on the lightweight wiring, but leaves wireLightweight's doc comment claiming the validate verbs need no config load
    fix: "Add `internal/loomcli#loomCLI.wireLightweight` to card 11's Edit group, after `internal/loomcli#loomCLI.validatePlanCmd`. Add an Intent bullet: `wireLightweight`'s doc comment, which says `validateDiscussionCmd` and `validatePlanCmd` read only paths and need no config load, now says that `validate-plan` loads the hub-wide `language.yaml` itself for its `rename-tool-missing` check, and that this path still loads none of `wire()`'s module configs."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/01-language-rename-tools.md:29
    summary: docs/overview.md's module-table line for internal/languageconfig names only the skills and the verify tiers, and no card adds rename_tools to it
    fix: "Add a `**Prosa:**` group to card 1, before `**Intent:**`, holding `docs/overview.md`. Add an Intent bullet: in `docs/overview.md`, the `internal/languageconfig/` line of the module tree also names the hub's structure-aware rename tools, which the Webster implementer prompt renders."
---

### [MEDIUM] Test-helper re-signs in cards 9 and 13 carry no arrow

**Location:** `_lyx/plan/09-webster-declared-changes.md:12`, `:14`; `_lyx/plan/13-implementer-rename-verify-markers.md:16`, `:17`

**Issue:** Card 9's Tests paragraph says `renderRecoveryWithUncommitted` and `renderMaster` gain a `patternChanges` argument.
Card 13's says `renderForkWithPack` and `renderRecoveryWithUncommitted` gain `renameTools` and `planVerify`.
Each of these is a parameter change on a function member listed under `**Edit:**`, but each glyph is bare.
The current signatures are at `internal/websterengine/template_test.go:348`, `:371` and `:377`.
The production renderers on the same cards carry their arrows, so only the test helpers are missing them.
`lyx loom validate-plan --overlay` over all four arrows together, plus F4's and F5's target additions, reports no blocking finding beyond the `{}` baseline, which has none.

### [MEDIUM] D5.4's bound is stated by no card

**Location:** `_lyx/plan/08-pattern-directive-changes.md:38`

**Issue:** D5.4 adds a route around `PATTERN.md`: a declared entry is followed and judged by its declared text, never the overview's current text.
The decision record bounds that route in its own `Bound:` sentence.
The bound limits it to entries the record's `## PATTERN changes` lists, in bullets of the D5.1 shape, appended after Discussion only through D5.2's caller guards, with D5.7 checking that the text lands.
Card 8 introduces the route by adding the fragment and its "judged by its listed new text" rule, but its Intent states no bound.
Cards 9, 10 and 17 also state none.
The other bounds in the record each reach a card's Intent: D1.4's in card 17, D5.2's in card 7, D2.5's in card 17 and D2.6's in card 13.
D5.4's is the only one that reaches no card.

### [MEDIUM] The Bouncer refusal row misstates the outcome

**Location:** `_lyx/plan/10-review-loom-declared-changes.md:39`, `:43`

**Issue:** Card 10 has `Bouncer.judgeCall` handle a malformed record through `degrade`.
`degrade` (`internal/shedadapters/bouncer.go:595`) returns `shedengine.Stuck` with a nil error.
In every review segment the Bouncer's `on_stuck` points at the segment's Burler-round row (`contracts/recipes/loom-recipe.yaml:215`, `:340`).
That row's `Engine.Review` then reads the same record, and the same card makes it return the error, which halts the run.
The run therefore halts at the Burler-round row with the Burler's message, not at the Bouncer.
The card's spec row records the Bouncer's own outcome as a correctness halt, which is not what the code does.

### [LOW] wireLightweight's doc goes stale

**Location:** `_lyx/plan/11-plan-gate-context-checks.md:58`

**Issue:** `internal/loomcli/wiring.go:255-269` says `validatePlanCmd` reads only paths and needs no config load, which is why it sits on `wireLightweight`.
Card 11 has `validatePlanCmd` call `languageconfig.Load` on that path.
Card 10 edits `wireLightweight` but writes before card 11's change exists, and card 11 does not list it.
After card 11, nothing corrects the comment.

### [LOW] docs/overview.md's languageconfig line omits the new key

**Location:** `_lyx/plan/01-language-rename-tools.md:29`

**Issue:** `docs/overview.md:270` describes `internal/languageconfig/` as the skills plus the verify tiers.
Card 1 makes `rename_tools` the module's third key, and card 13 renders it into the implementer prompt.
The decision record's Scope names the docs these changes touch, and no card updates this line.

## Rejected

- Card order (focus 2): every dependency follows its source card.
  Card 9 and card 10 use card 5's `DeclaredPatternChanges`.
  Card 11 uses card 10's `env.PatternRecordPath`, card 3's `SymbolRenamePairs` and card 4's `PatternChange`.
  Card 13's `RenderRecoveryPrompt` arrow builds on card 9's.
  Card 14 uses card 13's `RenameTools` deps field.
  Card 16 uses card 15's `CommitLogEntry`.
  No card depends on a later one.
- Reachability (focus 4): card 14 names its input and reaches it through `TestBeginBatch_Refusals`, `TestRecoverSpawnOrAttach` and `TestRenameToolMissingEnvelope`.
  Its input is a plan passing `planparser.Validate`, with a member-glyph old side and a `plan:` handle new side, under an empty list.
  Card 11 names inputs for `rename-tool-missing` and `pattern-changes-mismatch` and reaches both through `TestNewPlanGate` rows.
  `ValidateFormat` rejects neither input, since the overview has no section allowlist.
- Attack surface (focus 5): card 1's key and card 2's trailer gain their bounds where the license they serve is introduced: card 17's edit-directive exception and card 17's D2.5 rubric bullet.
  Card 7 states D5.2's bound in `decisionVerb`'s doc.
  Card 15 states that `departures` is informational and never changes a batch's status.
  The one gap is F2.
- Test economy (focus 6): the cards' new tests each pin a behavior no existing test covers.
  These are `TestSymbolRenamePairs`, `TestValidate_PatternChanges`, `TestDeclaredPatternChanges`, `TestRenameToolMissingEnvelope` and `TestVerifyGate_NotesFixCommitsOnce`.
  `TestRenameToolMissingEnvelope` pins a new single-flag envelope; `TestNoteRefusals_VerbEnvelopes` and `TestFingerprintRefusals_CarryPlanDrifted` cover other flags.
  No card plans one test per symbol or an assertion on static stencil prose.
- Scope of cards 12, 15 and 16 (focus 1): each traces to D4, D2.3/D2.4 and D2.8 respectively.
  Card 12's `TestDispatchLayers` edit follows from retiring the key that test uses.
- Card 7's edits to `orch-template-role.md` and `loom-template-arbiter.md` are not named in the record's Scope list.
  They tell the callers D5.2 admits that the form exists, where those stencils already name the verb, so D5.2 licenses them, and they are not flagged.
- Card 9's `hubgeom` import of `loomengine` for `DiscussionDecisionRecord` makes no cycle: `go list -deps ./internal/loomengine` holds no `hubgeom`.
- Card 10 has the loom tables derive the record path from `layout`.
  This does not break `PATTERN-told-geometry`, because `loomengine` is not a bound package, declares that path and already derives it from the same location in `DiscussionTable`.
- Verify coverage: the hub's tier list is `none`.
  `go test ./...` and `go test -tags integration ./...` run every targeted package's untagged and integration tests.
  The `tmux` and `llm` tags are vetted, and neither tier is hermetic.
