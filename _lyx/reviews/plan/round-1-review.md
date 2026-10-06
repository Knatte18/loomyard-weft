---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/03-tooling-contracts-prune.md:4
    summary: Card 3 still edits and prunes cmd/testtiming, which the parent decision "cmd/testtiming's tests are core's" removes from this task
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/00-overview.md:47
    summary: The skill-loading carve-out decision is carried by no card or Shared Decision, so card 14 and the prune procedure can fold, merge or delete TestSkillLoadSequence and TestSkillUnknown
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/18-rerun-measure-after.md:13
    summary: Card 18 reruns, keep-checks and splices every in-scope package including cmd/testtiming, whose tests this task may not edit, so its keep-directive check cannot be met
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/17-stencil-misc-prune.md:4
    summary: Card 17 bundles 21 unrelated packages into one commit, a catch-all rather than one independently reviewable unit
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/05-planglyph-prune.md:11
    summary: Card 5 cites a Shared Decision named scenario-merge that the overview does not define
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/00-overview.md:51
    summary: The prune procedure omits two Decision 13 bounds, the whole-scenario alone-rerun after a merge and the rule that cheap unit tests are not merged into scenarios
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/18-rerun-measure-after.md:13
    summary: Card 18 splices into docs/benchmarks/test-redundancy.md without stating the markdown-link-integrity constraint the decision record names for those edits
---

### [BLOCKING] Card 3 still prunes cmd/testtiming

**Location:** `_lyx/plan/03-tooling-contracts-prune.md:4`, `:16`, `:18`, `:22`; `_lyx/plan/00-overview.md:17`.
**Issue:** The decision record's parent addition "cmd/testtiming's tests are core's" says: "This task edits no file under cmd/testtiming, tests included. Card 3 drops cmd/testtiming from its package list."
Card 3 keeps `cmd/testtiming#` in its `Edit` targets, names the package in its Intent ("only its tests are pruned or folded"), and its `ImpactSummary` says "Test files of these packages only".
The overview's Card Index line 3 reads "prune cmd/testtiming, contracts and tools tests".
Core changes `cmd/testtiming`'s redundancy mode and tests in parallel, so this card plans the landing conflict the decision exists to prevent.
Card 18's `Uses: cmd/testtiming#` is not itself a defect: the card reads and runs the tool and does not change it.
It is not the only remaining touch, though; see F3.
**Fix:** Remove `cmd/testtiming#` from card 3's `Edit` list, Intent and `ImpactSummary`, and drop it from the Card Index line and the card's title wording.
Card 3 then prunes `contracts/specs`, `contracts/stencils` and the `tools/...` packages.
Do not add a replacement sentence about the tool beyond what the decision record says.

### [BLOCKING] The skill-loading carve-out is carried nowhere

**Location:** `_lyx/plan/00-overview.md:47` (`prune-procedure`), `_lyx/plan/00-overview.md:36` (`ownership-boundary`), `_lyx/plan/14-reed-shuttle-prune.md:10`.
**Issue:** The decision record's parent addition "Skill-loading tests are left to skills-in-one-turn" is a Decision: no test naming `Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill` or `loadSkillsThenPrompt` in `internal/shuttleengine/claudeengine` is edited, folded, merged or deleted, nor the `SkillLoader` fakes they use, and a redundancy candidate among them is kept with `//testtiming:keep` naming skills-in-one-turn.
A search of the plan directory for "skill" finds nothing.
The exposure is real: card 14 targets `internal/shuttleengine/claudeengine#`, whose `session_test.go` holds `TestSkillLoadSequence` and `TestSkillUnknown`, and the card calls the package's tests "fold leads".
The prune procedure's fold, merge and delete steps (4, 5 and 6) and the keep step have no exception for these tests.
The parallel task rewrites these tests, so a prune pass over them conflicts at landing and could delete a test that task changes.
**Fix:** Add the carve-out as a Shared Decision, or as a clause of `ownership-boundary`, applying to the cards that reach these packages (card 14 and any other that does), and state it in card 14's Intent.
It names the symbols the record names, says the tests keep their own names and are neither folded, merged nor deleted, and says a candidate among them gets `//testtiming:keep` naming skills-in-one-turn.
The `internal/loomcli` stub scripts in the record's list need no card, since that package is core's already.

### [MEDIUM] Card 18 treats cmd/testtiming as an in-scope package

**Location:** `_lyx/plan/18-rerun-measure-after.md:13`, `:14`, `:16`.
**Issue:** Card 18 reruns the redundancy report "once per in-scope package", requires every listed candidate to carry a `//testtiming:keep` directive, treats an uncommitted gap as a fix in "that package's test files", and splices each scratch file's section into the report.
`cmd/testtiming` is in the report and, under the decision record's Scope, not among the excluded six packages, but its test files are now off limits.
A candidate in its section can neither be annotated nor deleted by this task, and the card gives no instruction for that case.
**Fix:** State in card 18 that `cmd/testtiming` is excluded from the keep-directive check and from the section splice, because core regenerates its own section and owns its tests.
Keep `Uses: cmd/testtiming#` for running the tool.
Decide in the same sentence whether card 1 and card 18 still measure `cmd/testtiming` for the before and after table (reading only), and say so.

### [MEDIUM] Card 17 is a catch-all of 21 packages

**Location:** `_lyx/plan/17-stencil-misc-prune.md:4`.
**Issue:** Card 17 groups `internal/stencil`, `stencilcli`, `stencilstore`, `state`, `agentname`, `batcher`, `buildinfo`, `clihelp`, `cliwire`, `friction`, `frictionengine`, `idecli`, `ideengine`, `modelspec`, `pattern`, `quarrycli`, `selfreportcli`, `selfreportengine`, `shell`, `tokenvocab` and `vscode` under the title "remaining".
Its own Intent says `internal/stencil` carries a large candidate set, and `ideengine` and `cliwire` carry the card's scenario-merge leads, while most of the rest are small leaf packages.
The grouping is by leftover, not by concern.
The plan's resume mechanism is commit-per-card, so a half-done 21-package card is restarted from nothing, and a reviewer gets one diff spanning stencil, IDE, friction and leaf packages.
Cards 13 (nine packages of one review and gate family) and 16 (git, geometry and filesystem substrate, one family) are coherent groups and not a defect; this finding concerns card 17 only.
**Fix:** Split card 17 into at least two cards by family: the stencil packages with `state`, and the CLI, IDE, friction and leaf packages.
Renumber the later card (the rerun card becomes the last number), its file name, Card Index line and `Commit:` subject, and every `cards 2–17` reference in the overview.

### [LOW] Card 5 cites a Shared Decision that does not exist

**Location:** `_lyx/plan/05-planglyph-prune.md:11`.
**Issue:** The card points at "Shared Decisions, `scenario-merge`".
The overview defines two Shared Decisions, `ownership-boundary` and `prune-procedure`; the scenario rules live in `prune-procedure` step 4.
**Fix:** Change the reference to `prune-procedure`, step 4.

### [LOW] Two Decision 13 bounds are not carried

**Location:** `_lyx/plan/00-overview.md:51`.
**Issue:** Step 4 carries one tier per scenario, serial steps, the stop on a failing step, the state-sharing comment and the `t.Parallel` comment, and it correctly carries the three superseding parent decisions with no per-family timing and no restore bound.
It omits two bounds of Decision 13 that the parent additions did not supersede: after a merge, the alone-rerun of a `failed` entry is `-run '^<Scenario>$'` on the scenario as a whole, and cheap unit tests are not merged into scenarios and stay table-driven.
Step 2 gives the before-merge verdict but not the after-merge rerun, and a merge-happy package card could pull unit tests into a scenario.
**Fix:** Add both bounds to `prune-procedure` step 4 in one sentence each, worded as the decision record words them.

### [LOW] The link-integrity constraint is missing from card 18

**Location:** `_lyx/plan/18-rerun-measure-after.md:13`.
**Issue:** The decision record's Constraints require that "the edits to `docs/benchmarks/test-redundancy.md` keep every link resolving" (`PATTERN-markdown-link-integrity`).
Card 1 states it for its own page, but card 18, which splices into the report, does not.
**Fix:** Add one clause to card 18's splice paragraph that every link in the edited report and in `test-suite-prune-rest.md` still resolves.

## Fidelity sweep and other settled questions

Each is a verdict, not a finding:

- Scenario within one tier: carried by `prune-procedure` step 4 and by cards 14 and 15, which restrict merges to the `tmux` tier and keep `cli_tmux_test.go` out of an `integration` scenario.
- Steps sharing state, no restore bound: carried by step 4, with the comment on the relying step.
  Neither the plan nor any card re-introduces a restore condition; the one use of "restored" in card 18 is a lost coverage block brought back by a test, which is Decision 12.
- Merges judged by fixture builds, no per-family timing: carried by step 4 and the commit-message rule.
  The plan contains no per-family timing, median-of-three or spread condition at package level other than the Decision 11 and 12 measurements in cards 1 and 18, which the record requires.
- Measurements in a committed report linked from `running-tests.md`: carried by cards 1 and 18, with the link line placed beside `test-redundancy.md`'s and the before half committed first.
- Per-package cards 4 to 10: every file the cards name exists, every integration-tagged file they name carries the tag, and the `failed` entries in cards 2, 7, 9, 13 to 15 and 17 match the report's `failed` lines one for one.
  None plans a new test or a per-symbol test.
  The cards name only some of a package's tagged files (for example `battencli`, `landingshed`, `shedcli`) and say "hold the leads"; that is lead guidance, not a closed list, and not a defect.
- Verify coverage: the plan-level `## verify:` runs `go test` untagged, under `integration` and under `tmux` over `./...` and vets `llm`, which covers every targeted package.
  The `Verify:` lines on cards 13 to 15 are exceptional per-card additions that name packages the plan-level gate also covers; the `tmux` runs on `reedcli` and `webstercli` and the `llm` vets match the packages that carry those tags.
- Sizing of cards 13 and 16: each is one family, accepted.
- Writer/reviewer symmetry: the writer stencil does not tell the plan writer to omit anything this review flags as missing.
