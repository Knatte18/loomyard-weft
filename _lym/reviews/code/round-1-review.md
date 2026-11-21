---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: contracts/specs/loom-discussion-spec.md:337
    summary: The bug-record rule is unscoped, so every darn Record-Write record of a bug entry breaks it, though no writer can fill those sections.
    fix: "In contracts/specs/loom-discussion-spec.md, change the opening line of `### A bug entry's record` to scope it to a record the Discussion-Write lead writes, e.g. \"A record the Discussion-Write lead writes for a board entry labelled `bug` states up front what the red-first gate later checks:\". Keep the three bullets and the closing waiver paragraph unchanged."
    origin: lens:code-affected-set
---

### [MEDIUM] The bug-entry rule binds Record-Write records it cannot be met by

**Location:** contracts/specs/loom-discussion-spec.md:337

**Issue:** The new subsection sits under `## The decision record`, which governs every loom record, and opens "A record for a board entry labelled `bug` states up front…" with no row named.
Darn's Record-Write writes the record for exactly the runs the red-first gate guards (`contracts/recipes/darn-recipe.yaml:140`), and `BoardRecord` puts the board body fenced under `## Goal` and leaves `## Constraints` and `## Acceptance criteria` empty (`internal/loomshed/recordwrite.go:46-64`).
No agent writes that record, so it can never satisfy the rule, and the spec now states a contract that a whole class of records breaks.
The answer key scopes the change to "When Discussion-Write runs for an entry labelled `bug`" (`_lym/discussion/decision-record.md:9`); the spec's own citations subsection shows the convention of naming the row a rule applies to.
The assistant reviewer raised the same defect independently.

## Rejected

- code-correctness F1 (escape clause omits the gate's unjudged exit-5 case): the board entry names the two alternatives verbatim, "the test that will fail on the merge base and pass after the fix, or why no test seam exists", and the diff carries exactly them. Whether a fix touches a non-Go file is a write-time fact the darn writer's stencil already routes to the parent's `No test seam` entry (`contracts/stencils/darn/darn-template-write.md:110-113`); widening the Discussion rule goes beyond the answer key.
- Missing `Root-Cause:` trailer: the board entry is labelled `enhancement`, `stencil`, `loom`, not `bug`, and asks for a process change, not a bug fix, so no trailer is due.

Focus checks with no finding: Step 1's `lym board get` envelope carries the task's `labels`, so `bug` is detectable there, and the stencil paragraph adds no marker, conditional or engine term and leaves every other rule intact.
The spec's merge-base test and parent-only `No test seam` waiver match `internal/gatecli/redfirst.go` and `internal/loomshed/redfirstgate.go`, and the new subsection sits beside the other record rules under `## The decision record`.
The rubric bullet names the same three requirements as the spec, neither more nor fewer, and the stencil points at the spec instead of restating its placement.
The commit message and `_lym/landing/summary.md` claims match the diff; the `two-way` merge-danger call holds for a prose-only change to seeded stencils and a spec.
