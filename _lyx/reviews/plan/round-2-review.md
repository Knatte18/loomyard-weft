---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-implementer-verify-tiers.md:3
    summary: >-
      Card 4 re-signs the test helpers renderForkWithPack and renderRecoveryWithUncommitted without the re-sign arrow,
      and leaves out their wrappers renderFork, renderRecovery and renderForkPromptAndPack, which must change to compile.
    fix: >-
      In the Edit list of 04-implementer-verify-tiers.md, replace the line for internal/websterengine_test#renderForkWithPack
      with the same glyph followed by ` -> ` and the backticked signature
      `func renderForkWithPack(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, layout websterengine.PackLayout, pack []planindex.CardPack, verifyTiers []string) (prompt, packFile string, err error)`,
      and the line for internal/websterengine_test#renderRecoveryWithUncommitted with the same glyph followed by ` -> `
      and `func renderRecoveryWithUncommitted(batch batcher.Batch, repoRoot, anchorRoot, stencilsDir, specsDir, failureDigest, uncommittedPaths, notePath, parent string, verifyTiers []string) (string, error)`.
      Add three Edit lines, internal/websterengine_test#renderFork, internal/websterengine_test#renderForkPromptAndPack
      and internal/websterengine_test#renderRecovery, and extend the Intent sentence that says renderFork and renderRecovery pass nil
      to name renderForkPromptAndPack as passing nil to renderForkWithPack too.
      No change to the targets of any other card.
      Checked with lyx loom validate-plan --overlay, which reports no blocking finding for this card text and none for the baseline.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-rubric-verify-tiers.md:24
    summary: >-
      Cards 2, 3 and 4 let a verify_tiers entry skip a tier (Plan-Review accepts a compile-only tier, the plan section leaves it unrun, a raw run compiles it)
      but none of their Intents states what the entry can now let through and what bounds it, which the decision record gives for D5.
    fix: >-
      Add the D5 bound to the Intent of Card 3 (03-rubric-verify-tiers.md) as one sentence after its first bullet:
      an entry that calls a tier compiled rather than run lets a plan leave that tier unrun and Plan-Review accept it;
      the key is Guard-classified in a board-only file, so only the hub operator writes it;
      inside webster the Go-coded card and batch gates still run the untagged and -tags integration tests over the batch's packages whatever an entry says,
      and at landing the skip is bounded only by landing config's publish_verify;
      the shell guard and the plan gate refuse what they refuse whatever an entry says.
      Add one sentence to the Intent of Card 2 (02-plan-verify-tiers.md) saying the plan writer may leave a tier the list calls compiled unrun in its verify section, bounded as Card 3 states,
      and one sentence to the Intent of Card 4 (04-implementer-verify-tiers.md) saying a tier the list calls compiled is compiled, never run, by the implementer's raw runs only,
      while every rendered card or batch gate step still runs as written, which bounds it.
      Intent text only; no target, verify command or card number changes.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/03-rubric-verify-tiers.md:36
    summary: >-
      Card 3 updates the file header comment of rubric_test.go but not the doc comment of rubricMarkerAllowlist, which also enumerates the markers a rubric may carry and would go stale.
    fix: >-
      In the Intent of 03-rubric-verify-tiers.md, extend the bullet about rubric_test.go's file header comment so it also says
      the doc comment of rubricMarkerAllowlist, which lists the markers ReadRubric fills, names verify_tiers.
      The glyph contracts/stencils#rubricMarkerAllowlist is already an Edit target, so no target changes.
---

## Verdict

The plan carries every Decision (D1 to D7) and every Constraint of the decision record, in card order that holds, with a `## verify:` section that runs every targeted package's tests in the order the plan template states.
No card writes this task's own board entry, none adds a CLI command, and the new refusals in Card 1 each have a named test row reaching them.
The findings below are Intent and target-list gaps, none of which blocks the plan.

### [MEDIUM] Card 4 test helper re-signs carry no arrow and miss their wrappers

**Location:** `_lyx/plan/04-implementer-verify-tiers.md:3`
**Issue:** The Intent says `renderForkWithPack` and `renderRecoveryWithUncommitted` gain a `verifyTiers []string` parameter, yet both appear in the Edit list as bare glyphs, which the re-sign arrow rule flags.
`internal/websterengine/template_test.go` also has `renderFork`, `renderRecovery` and `renderForkPromptAndPack` calling them (`renderFork` at line 333, `renderRecovery` at line 360, `renderForkPromptAndPack` at line 350), so each must change to compile.
Only `renderFork` and `renderRecovery` are mentioned, in the Intent, and none of the three is a target; `renderForkPromptAndPack` is not mentioned at all.
The direct calls inside `TestRenderPrompts` are covered by that glyph being a target.

### [MEDIUM] No card states the bound on a tier skip

**Location:** `_lyx/plan/03-rubric-verify-tiers.md:24`
**Issue:** The decision record's D5 Bound says an entry can skip a tier and names what bounds it (the Guard class on a board-only file, the Go-coded card and batch gates before the parent merge, landing config's `publish_verify` at landing, the shell guard and plan gate unchanged).
Card 3 introduces "a tier the list says is compiled rather than run is not a finding" at Plan-Review, Card 2 lets the plan's `## verify:` section leave such a tier unrun, and Card 4 makes a raw run compile it, yet each Intent is silent on what it lets through and what bounds it.
The attack-surface rule requires the introducing card to state it.

### [NIT] The `rubricMarkerAllowlist` doc comment is not updated

**Location:** `_lyx/plan/03-rubric-verify-tiers.md:36`
**Issue:** `contracts/stencils/rubric_test.go` holds two comments listing the markers: the file header (line 2, which Card 3 updates) and the doc comment of `rubricMarkerAllowlist` (line 18, "only specs_dir, stencils_dir, publish_failure, webster_record and merge_review").
Card 3 edits the variable but its Intent names only the header, so the second list can be left stale.

## Rejected

- `TestHubGenericStencils_NameNoGoTiers` in Card 4 is not a static-prose pin: `PATTERN-test-economy` admits a rule across a stencil family, and the decision record's acceptance criterion on rendered prompts naming no Go tier command is that rule, shaped like `TestTwillStencils_HostNeutral`.
- Card 5 carries no separate bound sentence: its Intent already says a Decision-overturning finding is planned only once the record carries it and that a decline halts the run, which is the bound D1 states.
- The plan's `## verify:` section and the cards' packages: every targeted package is under `./...` for the untagged and `-tags integration` runs, and the `tmux` and `llm` tiers are compiled, so no verify-coverage finding.
