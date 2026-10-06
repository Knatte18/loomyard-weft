---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md:60
    summary: The prune-procedure's delete-or-keep step drops Decision 10's closed list of valid keep reasons and its never-annotate rule, so any candidate can be annotated away
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/18-cli-leaf-prune.md:1
    summary: Card 18 is a leftover grouping of fifteen packages from three unrelated concerns, not one independently reviewable unit
---

### [MEDIUM] Keep reasons are unbounded in the prune procedure

**Location:** `_lyx/plan/00-overview.md:60` (`prune-procedure`, step 6), with `_lyx/plan/19-rerun-measure-after.md:16`.
**Issue:** Decision 10 lists exactly four valid keep reasons: what the test pins that its covering tests do not, the core or testkit file that names it, the assertion it received and its source test, and, for a `Removable: no` candidate no fold or merge applies to, the covering test the report ties it to.
It adds: "A candidate with none of those reasons is deleted or folded, never annotated."
Step 6 says only that a candidate is "folded, merged or kept with `//testtiming:keep <reason>`", and card 19 checks only that the reason is non-empty.
Nothing in the plan states the four classes or the never-annotate rule.
The step 8 loop ("until the scratch list shows only candidates that carry a keep directive") and card 19's check are then satisfied by annotating every candidate with any text, and the prune would shrink nothing.
Step 6's "grouped by reason class" has no classes to group by.
Card 2 and card 14 each name one reason in passing, which does not bound the rest of the packages.
**Fix:** In step 6, list Decision 10's four reason classes, state that a candidate with none of them is deleted or folded and never annotated, and have card 19's check compare a directive's reason against those classes rather than only its presence.

### [MEDIUM] Card 18 is a leftover grouping

**Location:** `_lyx/plan/18-cli-leaf-prune.md:1`, with `_lyx/plan/00-overview.md` Card Index line 18 and the Shared Decisions' card ranges.
**Issue:** Card 18 collects fifteen packages (`batcher`, `buildinfo`, `clihelp`, `friction`, `frictionengine`, `idecli`, `ideengine`, `modelspec`, `pattern`, `quarrycli`, `selfreportcli`, `selfreportengine`, `shell`, `tokenvocab`, `vscode`) under "small CLI, IDE, friction and leaf packages".
Every other package card names one concern (parsers, logger, batten, board, loom, shed, stencil and state); this one is what remained after they were drawn.
It mixes CLI and engine pairs with IDE integration (`idecli`, `ideengine`, `vscode`, `quarrycli`, `selfreportcli`, `selfreportengine`, `clihelp`), a friction pair (`friction`, `frictionengine`) and pure leaf packages (`batcher`, `buildinfo`, `modelspec`, `pattern`, `shell`, `tokenvocab`) whose tests are import-allowlist and seam checks.
A reviewer of this card has no single theme to judge the prune against, and a run that fails partway leaves an unrelated mix half done.
**Fix:** Split card 18 into two cards: the CLI and IDE packages (`clihelp`, `idecli`, `ideengine`, `quarrycli`, `selfreportcli`, `selfreportengine`, `vscode`) and the friction and leaf packages (`batcher`, `buildinfo`, `friction`, `frictionengine`, `modelspec`, `pattern`, `shell`, `tokenvocab`).
The new second card becomes card 19 and the rerun card becomes card 20, with the Card Index, file names, `Commit:` subjects, the `Applies to:` ranges and the "card 19" references in card 1 and the rerun card updated to agree.

### Points settled without a finding

- `cmd/testtiming`: no card edits, folds, merges or deletes a test there or names it in an `Edit` target.
  The `ownership-boundary` decision and card 3 state the exclusion, card 18 does not mention the package, and card 19 lists it only under `Uses:` and carves it out of the keep check, the gap fix and the splice, consistent with the decision record.
- Skill-loading tests: `ownership-boundary` carries the full symbol list and the `skills-in-one-turn` keep reason, and card 14 is the only card that reaches `internal/shuttleengine/claudeengine`.
  The only tests in that package naming those symbols are `TestSkillLoadSequence` and `TestSkillUnknown`, which card 14 names.
  The `SkillUnknown` method on `fakeSession` in `internal/orchengine` and the `Spec.Skills` assertions in `internal/orchcli` and `internal/loomengine` sit outside the decision record's named directories, so the plan is right not to extend the carve-out to them.
- Shared Decision names: the only names cards cite, `prune-procedure` and `ownership-boundary`, are both defined in the overview.
  Step 4 carries the whole-scenario alone-rerun and the no-merge-of-cheap-unit-tests bounds.
- Package coverage: every in-scope `## ` section of `docs/benchmarks/test-redundancy.md` is a target of exactly one card or, for `cmd/testtiming`, of card 19's read-only measurement; no core or `internal/testkit` package is targeted.
- Card Index, file names, headings, `Commit:` numbers and the `Applies to:` ranges agree.
