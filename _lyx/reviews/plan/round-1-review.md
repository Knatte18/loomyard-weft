---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/01-language-verify-tiers.md:62
    summary: Card 1's new TestWireHub_LanguageSkills verify_tiers refusal row duplicates the existing EntryWithLeadingSpaceIsRefused row plus Card 1's own TestLoad_InvalidFailsNamingKey verify_tiers rows (PATTERN-test-economy)
    fix: In Card 1, delete the Tests bullet that starts with `TestWireHub_LanguageSkills` gains a row whose file holds an empty verify_tiers entry (line 62), and delete the target line `- internal/webstercli#TestWireHub_LanguageSkills` from Card 1's Edit list (line 15). The loom TestWire_LanguageSkills verify_tiers row (line 61) stays as the one real spawn path the acceptance criterion asks for. Card 4 keeps its own TestWireHub_LanguageSkills edit. Checked with lyx loom validate-plan --overlay; it reports no new finding.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/01-language-verify-tiers.md:35
    summary: Card 1 prescribes a Load branch for an absent verify_tiers node that no input can reach, since configengine fills every template key the file lacks before Load sees the document
    fix: In Card 1's Intent, replace the sentence starting `Load` runs it for `skills` as today, and for `verify_tiers` only when the node is present with this. Load runs stringList over both keys unconditionally. An existing hub file without verify_tiers reaches Load holding the template's empty verify_tiers list, because configengine's load fills missing template keys through yamlengine.FillMissing, so it loads unchanged. Keep the TestLoad_ResolvesSkills row with a skills list and no verify_tiers key, which now pins that template fill.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-plan-verify-tiers.md:39
    summary: Cards 2 and 3 assert the file's tiers on TestWire_LanguageSkills's listed row, which Card 2 says Card 1 gave tiers, but Card 1 never adds verify_tiers to that row or a want column for it
    fix: In Card 1's Tests bullet for TestWire_LanguageSkills (01-language-verify-tiers.md line 61), add that the listed row's file also holds a two-entry verify_tiers list, and that the row struct gains a wantTiers column holding those entries, which Card 1 asserts against the Language that loomCLI.wire keeps. In Card 2 line 39 and Card 3 line 46, compare against the row's wantTiers column instead of saying the file's tiers.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-implementer-verify-tiers.md:58
    summary: Card 4 has renderForkWithPack and renderRecoveryWithUncommitted pass nil tiers, yet its TestRenderPrompts rows render with two tiers through no route the card names, since every TestRenderPrompts row goes through renderFork and renderRecovery, which call those helpers
    fix: In Card 4 line 58, give renderForkWithPack and renderRecoveryWithUncommitted a verifyTiers []string parameter that they pass to RenderForkPrompt and RenderRecoveryPrompt. Have TestRenderPrompts' two-tier rows call those two helpers directly with the tiers. renderFork, renderRecovery and TestRenderPrompts_ParentDirective's direct calls pass nil. No target is added, since both helpers are already Card 4 targets.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/01-language-verify-tiers.md:49
    summary: Card 1 edits loomCLI.wire but leaves its comment The error already names skills and its own way forward, which goes stale once Load's error names whichever key failed
    fix: In Card 1's Callers bullet, add that the comment above loomCLI.wire's languageconfig.Load call (internal/loomcli/wiring.go, inside loomCLI.wire) says the error already names the failing key and its own way forward.
---

### [BLOCKING] Redundant webster wire row for the verify_tiers refusal

**Location:** `_lyx/plan/01-language-verify-tiers.md:62`

**Issue:** `websterCLI.wireHub` passes every `languageconfig.Load` error through `configLoadError("language.yaml", err)` (`internal/webstercli/wiring.go:149`), whatever key failed.
`TestWireHub_LanguageSkills`'s existing `EntryWithLeadingSpaceIsRefused` row already pins that wrapping: invalid `configLoadRefusal`, naming `language.yaml` and `lyx config language`.
Card 1's own `TestLoad_InvalidFailsNamingKey` rows pin that a bad `verify_tiers` value fails naming `verify_tiers`.
Together they cover the new webster row's behavior.
The acceptance criterion needs one real spawn path, and Card 1's loom `TestWire_LanguageSkills` row already provides it.
Under `PATTERN-test-economy`, a redundant test is a finding exactly as a missing one is, so the row is BLOCKING.

### [MEDIUM] Unreachable absent-node branch in Load

**Location:** `_lyx/plan/01-language-verify-tiers.md:35`

**Issue:** `languageconfig.Load` reads the file through `configengine.LoadOrTemplate`.
Its shared `load` body (`internal/configengine/config.go:196`) fills every template key the stacked document lacks through `yamlengine.FillMissing`.
Card 1 adds `verify_tiers: []` to the template, so an existing hub file without the key reaches Load's `yaml.Unmarshal` with the key present as an empty sequence.
An absent file degrades to the template itself, and an empty board dir returns before any read.
The node-kind-zero branch therefore has no input that reaches it.
The test row the card means to cover it, a skills list with no `verify_tiers` key, passes through the template fill and never reaches the branch.
That is a code path for a state the upstream fill rules out.

### [MEDIUM] Listed wire row never carries tiers

**Location:** `_lyx/plan/02-plan-verify-tiers.md:39`

**Issue:** Card 2 says "Card 1 put tiers in its file" for `TestWire_LanguageSkills`'s listed row, and Card 3 (line 46) asserts that row's `c.env.VerifyTiers` against "the file's tiers".
Card 1's `TestWire_LanguageSkills` bullet (line 61) adds only a `wantKey` column and a new error row.
The listed row's content stays `skills:` alone (`internal/loomcli/wiring_test.go:1025`).
As planned, both later assertions compare an empty list against an empty list and pin nothing.

### [LOW] No route for Card 4's two-tier render rows

**Location:** `_lyx/plan/04-implementer-verify-tiers.md:58`

**Issue:** Every `TestRenderPrompts` row renders through `renderFork` or `renderRecovery` (`internal/websterengine/template_test.go:660`, `:667`).
Those call `renderForkWithPack` and `renderRecoveryWithUncommitted`, which Card 4 line 58 has pass nil tiers.
The card's two-tier rows (line 59) then have no named way to hand tiers to the renderer.
An implementer must invent one, either 17-argument direct calls or a new helper parameter.

### [NIT] Stale comment on the loom wire's Load call

**Location:** `_lyx/plan/01-language-verify-tiers.md:49`

**Issue:** `internal/loomcli/wiring.go:503` reads "The error already names skills and its own way forward."
After Card 1, the error names whichever key failed, which can be `verify_tiers` or no key at all.

## Rejected

- Card 4's `TestHubGenericStencils_NameNoGoTiers` is not a defect.
  `PATTERN-test-economy` allows a stencil test that pins "a rule across a stencil family", and this scan pins one negative rule across the hub-generic plan, Plan-Review and implementer stencils, carried by `//lyx:guard` like `TestTwillStencils_HostNeutral`.
  `TestTwillStencils_HostNeutral` covers only `twill-` stencils (`contracts/stencils/govalues_test.go:74`), so it does not cover this rule.
  A grep shows these three stencils are the only registered stencils naming the banned strings, and every line naming one is rewritten by Cards 2, 3 or 4.
- The other `verify_tiers` pass-through rows on Cards 2–4 each pin a distinct edge.
  `TestPlanTable`, `TestReadRubric` and `TestRenderPrompts` pin the stencil fill not parsing `{{`.
  The Bouncer seed and judge test, `TestBouncerEntry_HappyPath` and `TestBurlerReviewEntry_RubricStencil` pin `BouncerConfig`→notes, `Env`→`BouncerConfig` and `Env`→Burler notes.
  `TestBeginBatch_PromptFile`, `TestRecoverBatch_UncommittedPathsPrompt` and the two wire tests pin deps→renderer and config→deps.
  None duplicates another, apart from F1.
- Fidelity: D1 → Card 5 (rework and rubric halves); D2 and D3 → Card 6; D4 → Card 2 (plan) and Card 5 (rework); D5 → Card 1; D6 → Cards 2, 3, 4 and 7; D7 → Card 1's `doc.go` with the four example bullets.
  Each Constraint is carried, and each Acceptance criterion maps to a card's tests or prose.
  No card adds scope the record does not license.
  `docs/overview.md` and `refusal-spec.md` follow from the repo's docs rule and `PATTERN-refusal-way-forward`, and `websterCLI.beginDeps` is the seam that makes the D6 value-map edge testable.
- Card order holds.
  Cards 2–4 use only Card 1's `VerifyTiersValue` and `Language`, and Card 4's scan reads stencils that Cards 2 and 3 have already edited.
  Card 5 edits the rubric's `Fidelity` bullet after Card 3's `Verify coverage` edit, adding no banned string, and Card 7's spec is outside the scan.
- Verify coverage holds.
  The repo has one module apart from an untargeted testdata module.
  The section runs untagged and `-tags integration` tests over `./...`, which covers `contracts/stencils` and `contracts/specs`, vets the `tmux` and `llm` tiers, and runs them in the stated order.
- Reachability holds for the refusals Card 1 prescribes.
  A line-break entry (`["a\nb"]`, `["a\rb"]`) passes `stringList`'s trim check and reaches the new refusal.
  The empty-key `unusable` form is reached by the unparseable-yaml row through configengine's `ErrInvalid` branch.
  A local override holding `verify_tiers` reaches `ValidateLocal`'s category refusal, which names the key, once the template carries the key.
- Card 5's stop route states its bound, as D1 settles it: the stopped rework writes no `coverage.md`, so the missing output halts the run for the parent.
