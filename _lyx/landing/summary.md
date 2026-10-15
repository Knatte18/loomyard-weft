# Webster deviations list stops naming a batch's own target files

A Webster fork's deviations list reported members of files and packages that a card targeted, because `planglyph.ScopeGuard` matched a touched symbol only by exact match against the target union.
The implementer prompt also asked the fork to compare its edits against the targets while telling it not to, so the fork had no single rule to act on.
This change gives the guard file and package coverage and gives the prompt one rule that states what the guard counts.

## What changes

- `planglyph.ScopeGuard` treats a touched symbol as in scope when a card's file target is the file it lives in, or a card's package target (a package self glyph or a directory path) is the directory its file lives in directly.
  A subdirectory of a package target is not covered, matching the bound the guard's symbol-less file check already has.
- A member glyph or member `plan:` handle target still covers only that member, never its package's other members.
- A Modified symbol is covered by file and package targets only when every file on its `Before` and `After` sides is covered, and a Renamed pair only when both `From.File` and `To.File` are covered.
  A symbol moved from a targeted file into an untargeted one is still reported.
  A Modified symbol with no `Before` or `After` file is still reported.
- Exact-target matching, rename-with-either-side-targeted and the symbol-less changed-file check keep their verdicts.
  The guard stays informational.
- `internal/planparser` exports `RefMember`, the former `refMember`, so `ScopeGuard` tells a member ref from a package self glyph or directory path through the planparser's classification instead of a new shape test in `planglyph`.
  A nil plan reads as the default language.
  `RefScope` cannot make that distinction and `RefCovers` reaches subdirectories, so neither fits.
- `contracts/stencils/webster/webster-body-implementer.md` states the deviation rule once, in per-card step 2: a member target covers that member, a file target every member of its file, a package target every member of the files directly in its directory, and a rename is inside when either name is a target.
  `**Uses:**` entries add no coverage.
  Three deviations are sanctioned, each within the bound the body already gives it: the guard-step fix, the scan-error fix and a fix to a failure the card caused.
  The guard records every deviation, the fork lists and compares none, and a deviation alone never makes the report `FAILED`.
  The guard-step paragraph's closing "every other deviation rule stands" now points at that rule.
  The fork and recovery prompts both compose this body, so both get the rule.
- `contracts/specs/loom-plan-spec.md`: the deviation union paragraph drops its description of a `UnitPath`-mapped file comparison the guard never made and points at `planglyph.ScopeGuard`, keeping the facts that the mismatch is informational and that `Uses:` is excluded.

## Design decisions

- The stencil states the rule instead of pointing at Go, because the fork never reads `ScopeGuard`'s doc comment.
  The plan spec points at the guard, since a pointer cannot go stale.
- `ScopeGuard`'s tests are extended rather than added to, and no test pins stencil prose.
  The `internal/planparser` test budget in `cmd/lyx/testdata/test-budget.yaml` rises by one for the `RefMember` test.
- Out of scope: the fork's report shape, `record-batch` and `recover-batch`, the Master template's `deviations` wording, `webster-body-verify-fix.md` and how the `weaver` note's implementer records a deviation.

## To check by hand

- Neither `lyx gate test ./...` nor `lyx gate test --tags integration ./...` is recorded as run for this change; run both before pushing.
- Read the rendered fork prompt and the rendered recovery prompt once each, and confirm step 2 states the rule once, names the three sanctioned deviations, and that neither prompt asks the fork to list or report deviations.
- `contracts/specs/loom-plan-spec.md` may conflict with the in-flight `card-own-entry-edit` task; the paragraphs differ, so a merge there should be clean.
