---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: contracts/stencils/drift/drift-template-baseline.md:22
    summary: The board entry asks for the docs-in-the-same-commit list in the drift baseline stencil; the diff only swaps the pointer, on the decision record's own D2 claim that no amendment backs
    fix: Leave the stencil as it stands and defer to the run's parent, because recording or refusing the D2 deviation with `lym loom decision add` lies outside the fixer's write surface. If the parent refuses it, add the docs and PATTERN rule to step 1 of drift-template-baseline.md in the generic wording of darn-template-write.md step 4.3.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/orch/orch-template-role.md:188
    summary: The new placement sentence names CLAUDE.md, though D4's acceptance criterion is that the orch role stencil names no CLAUDE.md
    fix: In orch-template-role.md line 188 reword the clause to "and lym adds no instruction file to a repository", dropping "such as `CLAUDE.md`".
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-parent-directive.md:13
    summary: The background file still calls the edit-directive stencil the only statement of the edit rule, though it now states a second rule, the own-worktree rule
    fix: In pattern/PATTERN-parent-directive.md line 13 change "the only statement of the edit rule" to "the only statement of the edit directive's rules".
---

### [MEDIUM] Drift baseline gets no docs rule the board entry asked for

**Location:** `contracts/stencils/drift/drift-template-baseline.md:22`
**Issue:** The board entry's first scope bullet moves the docs-in-the-same-commit list into `darn-template-write.md` step 4.3 and `drift-template-baseline.md`.
The diff changes the drift line to a README pointer and adds no docs rule, following the record's D2.
D2 is the writer's own claim, not an `Added after Discussion` entry, and the change description lists it as a design decision, not as an unrecorded parent correction.
The reason is sound, since the baseline agent changes no code, but the omission rests on no amendment to the entry.

### [LOW] The orch stencil still names CLAUDE.md

**Location:** `contracts/stencils/orch/orch-template-role.md:188`
**Issue:** The added sentence reads "lym adds no instruction file such as `CLAUDE.md` to a repository".
D4's acceptance criterion says the orch role stencil names no `CLAUDE.md`.

### [NIT] PATTERN-parent-directive background is one rule behind

**Location:** `pattern/PATTERN-parent-directive.md:13`
**Issue:** The line says the `edit-directive` stencil is the only statement of the edit rule.
The stencil's own header now reads "its rules' only statement", since it carries two rules.

## Rejected

- Orch rules file absent from the diff (focus item 2): the file lives in the hub's `_board`, outside this repository's diff, and the change description records that the parent wrote it through `lym orch rules`. I read `_board/_lym/orch/rules.md`: its Production section states the rule with the exception and a "Pushing `main` by hand" section carries the gate, so the omission is recorded and D3 holds.
- `merriamFixedContext` comment and the vocabulary scan (focus item 4): the value is unchanged, the comment is accurate (59 + 229 + 104 = 392), and the scan set and `PATTERN-fabric-vocabulary.md` drop `CLAUDE.md` together. Not defects.
- Remaining `CLAUDE.md` mentions in `merriambase_test.go`, `webstercli`, `runlevel_test.go`, `claudeengine/capabilities.go` and `internal/lenses/template.yaml` (focus item 3): they concern the engine's instruction-file handling for other repositories, and the record lists the capabilities file as out of scope. Not live citations of this repository's file.
- Test economy (focus item 5): the diff adds no test and no assertion on stencil prose, and the code-tests lens found no uncovered behaviour.
