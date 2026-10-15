---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 (last paragraph) and Scope (Discussion lens directive's relaunch wording)
    summary: D1 says the relaunch wording names all three outputs, but the relaunch part is a host-neutral twill stencil that names no file and says finish the reviewed files, which D1 itself keeps to two, and the Discussion host's spawn point still says both output files
    fix: In D1, replace the sentence about the relaunch wording with an explicit mechanism and bound. Recommended - the Discussion host's twill values (discussionLensDirective in internal/loomengine/twill.go) change SpawnPoint from both output files to all three output files per Step 5, and twilldirective gains one host-supplied value naming the files a relaunched writer finishes, rendered by contracts/stencils/twill/twill-relaunch.md in place of the reviewed files, with ReviewedFiles unchanged so the forks still review two files and the Plan host passes its own reviewed files for the new value. Add internal/twilldirective and contracts/stencils/twill/twill-relaunch.md to Scope In. If instead no relaunch change is wanted, say in D1 that the relaunch's per Step 5 reference carries the third file and drop the relaunch wording from Scope.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 Rationale (existence is enforced by D1's Outputs)
    summary: D3 leans on Outputs for existence but does not state that the writer's own Step 6 self-check passes with relevant.md missing, so a skip surfaces only as a held turn end, never as a gate finding
    fix: Add to D3's Bound line - a writer that skips relevant.md passes lyx loom validate-discussion and is caught by shuttle's held turn end, which notifies the parent on an autonomous run and holds an interactive run for the operator, bounded by the Discussion timeout; no gate re-prompt names the missing file, since the gate runs only once every output exists.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 Decision (never a reason to search further) against D4 Decision (then explore further as needed)
    summary: The spec clause D2 pins has no subject, so a Plan writer reading the spec can take never a reason to search further as binding on itself, against D4's explore further as needed
    fix: In D2, reword the clause to say the Discussion writer records only what it already read and never searches further to fill the list; leave Plan's further exploration to D4. In D2's Rationale, change no extra search for Plan to no extra Discussion search on Plan's behalf.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D3 Decision (absolute or contains ..) and D3 Bound
    summary: The bound claims the path rule keeps an entry inside the worktree, but absolute is left to the host's notion, so a ~/ path, and a drive-letter or UNC path checked on Linux, pass the check
    fix: In D3, state the path rule independent of the host OS - a finding for a path that starts with /, a backslash, ~ or a drive letter and colon, or that has a .. segment - or weaken the Bound to say the rule refuses the common outside forms and the list stays advisory. A tracked symlink inside the worktree needs no rule, since it is repository content the list does not introduce.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 and D4 (code head start) against the discussion spec's optional eighth section line
    summary: The spec still calls Notes for the plan writer a non-exhaustive head start in which the plan writer explores the codebase itself, while D4 makes relevant.md the code head start and D6 keeps Notes to design hints, and neither Decision rewords that line
    fix: In D6, add that the spec's description of the optional Notes for the plan writer section is reworded to design hints for the plan writer, never a completeness requirement, with the head-start wording living only in the relevant.md section, and that a hint may still name the symbol it is about, so no constraint the plan writer needs is pushed out of the record.
---

### [MEDIUM] Relaunch wording has no carrier for the third file

**Location:** `_lyx/discussion/decision-record.md` D1, last paragraph; Scope In, the relaunch item.
**Issue:** `contracts/stencils/twill/twill-relaunch.md` is host-neutral by its own header ("names no file, step, verb or section of any one host") and tells a relaunched writer to finish "the reviewed files" per `{{.format_step}}`.
For Discussion that phrase is `ReviewedFiles` in `internal/loomengine/twill.go`, which D1 keeps to the decision record and the support log, so the relaunch cannot name all three outputs without a new twilldirective value or a reliance on Step 5 that D1 does not state.
The same host's `SpawnPoint` reads "Once you have written both output files per Step 5", which D1 and Scope do not mention.
The shared stencil also serves the Plan host, so the choice is not Discussion-local and belongs in the record.

### [LOW] The enforcement story omits that the self-check passes a skipped file

**Location:** `_lyx/discussion/decision-record.md` D3 Rationale.
**Issue:** Checked against `internal/shuttleengine/wait.go`: a turn end without every output file is held, never terminal, and an autonomous run's held turn end notifies the parent once; the gate runs only on a Done, which needs every output.
So a skipped `relevant.md` is caught, and D1 and D3 are consistent, but the writer's own `lyx loom validate-discussion` reports clean while the file is missing.
D3 should state that bound so the plan writer does not add an absence finding to the gate, which would never fire for Discussion-Write and would fail the fix row.

### [LOW] D2's "never a reason to search further" reads as binding Plan

**Location:** `_lyx/discussion/decision-record.md` D2 Decision and Rationale; D4 Decision.
**Issue:** D2 pins spec wording that the list "is never a reason to search further" without saying whose search.
The spec is read by the Plan writer through the Plan stencil's pointer, and D4 tells Plan to explore further as needed.
The intended reading (no extra Discussion search to build the list) is recoverable from the Rationale's "no extra search for Plan", but that phrase is itself ambiguous.

### [LOW] The path rule's bound overclaims

**Location:** `_lyx/discussion/decision-record.md` D3 Decision and Bound.
**Issue:** "Absolute" is undefined; Go's `filepath.IsAbs` on Linux passes `C:\x` and `\\host\x`, and no rule names `~/`.
The project runs on Windows through WSL, so a list written there can carry those forms.
The stakes are small (the list is advisory and written by a trusted agent in the same run), but the Bound states containment the check does not deliver.

### [LOW] Two "head start" descriptions after D4 and D6

**Location:** `_lyx/discussion/decision-record.md` D6 and D4; the spec's eighth section line in `contracts/specs/loom-discussion-spec.md`.
**Issue:** D4 rewrites the spec's "sole input" sentence and D6 adds a rule placing the code list in `relevant.md`, but the optional section's own parenthetical, "a non-exhaustive head start for the plan writer … the plan writer explores the codebase itself", is left as is.
That leaves two sections described as Plan's head start.
Per the completeness-before-leanness test, D6 loses no constraint as long as a design hint may still name its symbol; stating that keeps "design hints only" from being read as forbidding code names in a hint.

## Rejected

- Focus on D3's present-empty versus absent asymmetry: not a defect.
  D3's Rationale states why absence passes (in-flight runs, the fix-row gate), and the second Auto-mode assumption states why an empty list is refused: every loom Discussion reads some file. A legitimate empty list is deliberately refused, with its reason.
- Focus on D2 against the Discussion stencil's Step 2 bound: not a defect.
  Step 2 forbids gathering signatures, `file:line` citations, interface shapes and dependency lists; D2 records only the file paths and member names the writer already read, and asks for nothing more.
- Focus on Scope against Decisions and Constraints: no requirement lives only in the support log.
  The doc-comment items live in Scope, which is part of the decision record Plan reads, and CLAUDE.md's docs rule binds them; the relaunch item is F1.
  Wiring the new path into the Plan and Discussion table values and the gate env is call-site enumeration, which the rubric's `Do not flag` list excludes.
- Focus on symlinks under D3's path rule: a tracked symlink is repository content, not something the list introduces; folded into F4's fix as needing no rule.
