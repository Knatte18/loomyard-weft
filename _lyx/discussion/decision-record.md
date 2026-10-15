# Webster fork deviations list mixes in its own target files

## Goal

Make Webster's deviation list name only changes outside the batch's card targets, and give the implementer prompt one rule for what counts as a deviation, the rule `planglyph.ScopeGuard` checks.

## Scope

In:

- `planglyph.ScopeGuard`: a touched symbol is in scope when a file or package target covers it, not only when it is itself a target.
- `contracts/stencils/webster/webster-body-implementer.md`: one deviation rule, stated once, and the dangling "every other deviation rule stands" clause reworded to point at it.
- `contracts/specs/loom-plan-spec.md`: its deviation union paragraph points at `planglyph.ScopeGuard` instead of describing a `UnitPath`-mapped file comparison the guard does not make.

Out:

- The fork's report shape, `record-batch`, `recover-batch` and the digest: the fork already reports only `status` and `head_sha`, and the digest's `deviations` already comes from the guard.
- The Master template's `deviations` wording.
- `webster-body-verify-fix.md`: the verify-fix fork has no card targets and no scope guard runs over it.
- Finding F12 of the stencil intent audit, which stays with the `stencil-intent-audit` note.
- How the `weaver` note's implementer records a deviation; this task decides Webster's only.

## Decisions

### D1: File and package targets cover their members in `ScopeGuard`

Decision: `ScopeGuard` treats a touched symbol as in scope when any of these holds:

- the symbol is a card target, a `plan:` handle counting as the glyph it stands for, as today;
- for a rename, either side is a target, as today;
- a card's file target is the file the symbol lives in;
- a card's package target, a package self glyph or a directory path, is the directory the symbol's file lives in directly.

A member glyph target still covers only that member; it never covers its package's other members.
The file-level check for a changed file with no symbols in the delta is unchanged.
The guard stays informational.

Rationale: today a symbol is in scope only by exact match against the target union, so a card that targets a file or package and edits members in it gets every one of those members, and so their files, reported as deviations.
That is the bug the board entry names, now produced by the guard rather than the fork.
The bound is the target's own extent: a file target covers that file, a package target that directory and not its subdirectories, matching the symbol-less file check's existing bound.

### D2: One deviation rule in the implementer body's per-card step 2

Decision: step 2 of the per-card loop, which already says to make exactly the changes the card describes in exactly the targets its type labels name, carries the rule:

- A deviation is a change outside the batch's card targets as `ScopeGuard` counts them: a member target covers that member, a file target every member of that file, a package target every member of the files directly in its directory, and a rename is inside when either name is a target.
- `**Uses:**` entries are never targets: a change to a member a card only reads is a deviation.
- Three deviations are sanctioned, each with the bound the body already gives it:
  the guard-step fix in step 3, only in a file the guard step's failure names;
  the scan-error fix the guard-fix line names, only in that file and under the card it names;
  and a fix to a failure the card caused, under the bounded self-fix section's two-pronged test and its attempt cap.
  No other change outside the targets is made.
- The scope guard records every deviation, sanctioned ones included; the fork lists and compares none, and a deviation alone never makes the report `FAILED`.

The guard-step paragraph's closing clause "every other deviation rule stands" becomes a pointer to step 2's rule.
The prior-batch paragraph's sentence about `scope-guard deviations` stays.

Rationale: the fork never reads `ScopeGuard`'s doc comment, so the body states the rule the fork acts on rather than pointing at Go; the fork and recovery prompts both compose this body, so the rule reaches both once.
The sanctioned exits already exist in the body and the guard-fix fragment; naming them as deviations removes the reading that they break a separate rule.

### D3: The plan spec points at `ScopeGuard`

Decision: the plan spec's deviation union paragraph, under its `## Deferred / forward-compat` heading, keeps its two facts, that the mismatch is informational and never blocking on its own and that `Uses:` is excluded, and replaces its description of the comparison with a pointer to `planglyph.ScopeGuard`, merged with the paragraph's existing pointer to `internal/websterengine`'s package doc.

Rationale: the paragraph's comparison does not match `ScopeGuard`, and a pointer to the code cannot go stale.

## Constraints

- `PATTERN-stencil-ownership`: edit `contracts/specs/loom-plan-spec.md` and the stencil under `contracts/stencils/` only; the deployed copies under the board refresh through the normal deploy, never by hand.
- `PATTERN-producer-pointer-rule`: the plan spec points at `ScopeGuard`; the stencil's rule is the fork's instruction, stated because the fork cannot read the guard.
- `PATTERN-ref-shape-registry`: telling a member glyph from a package self glyph, a file path or a directory path goes through `internal/planparser`'s classification, never a new shape test in `planglyph`.
- `PATTERN-test-economy`: `ScopeGuard`'s coverage extends its existing tests; no test pins stencil prose, and the Master digest-field set the webster template test pins stays as it is.
- `ScopeGuard`'s doc comment states the new coverage, one sentence per line.
- Semantic line breaks in both markdown files.

## Auto-mode assumptions

- No operator answered; the picks are in the support log's question ledger.

## Open risks

- `card-own-entry-edit` is running and may also edit `contracts/specs/loom-plan-spec.md`; the paragraphs differ, so a Merge-In there is expected to be clean.

## Acceptance criteria

- A symbol in a card's file target is not reported by `ScopeGuard`.
- A symbol in a file directly in a card's package target directory is not reported; one in a subdirectory is.
- A symbol in the same package as a member glyph target, but not that member, is still reported.
- A `plan:` handle target, a rename with either side targeted, and a symbol-less changed file keep today's verdicts.
- The rendered fork prompt and the rendered recovery prompt each state the deviation rule once, in per-card step 2, naming the three sanctioned deviations with their bounds and saying a deviation alone never makes the report `FAILED`.
- Neither prompt asks the fork to list, compare or report deviations, names a glyph or changes-files union, or refers to a deviation rule it does not state.
- The plan spec's deviation paragraph names `planglyph.ScopeGuard` and no longer describes a `UnitPath`-mapped comparison.

## Notes for the plan writer

- `targetsCover` already maps a target to a file or package directory through `planparser.RefScope`, but it counts a member glyph's package directory too; the symbol check must not.
- The fork and recovery prompts both compose `webster-body-implementer.md`, so the recovery path needs no separate stencil edit.
