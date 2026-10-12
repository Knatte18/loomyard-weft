# Card 6 — plan-review-rubric-overlay

**Edit:**
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Make the Plan-Review reviewer check its proposed fixes against the plan gate before writing them, and tie a comment-bound glyph to its enclosing function (Decisions 6 and 7, issue #563).
The edit lands in `contracts/stencils/loom/`, never the hub's deployed copy (`PATTERN-stencil-ownership`).

In `## Proposed fixes`, the rubric gains the overlay directive:
before writing a finding whose `fix` adds, drops, moves or pairs targets, or changes a verify command, the agent writing the review file runs `lyx loom validate-plan --overlay` once with every such fix applied together, its Bash timeout raised to cover a type load, after a baseline run of `validate-plan --overlay` with `{}`, never the flag-less run, so no run writes a file;
it proposes no fix whose overlay run reports a blocking finding the baseline does not.
When the combined run reports a new blocking finding, it re-runs with each fix overlaid alone; a fix whose solo run reports a new blocking finding is withheld, and its defect is reported with the constraining check named, as the section already prescribes for a defect with no admissible fix.
A new blocking finding of the combined run that no solo run reproduces is a joint conflict: the fixes that finding names by card are withheld and their defects reported together the same way.
A fix that inserts, removes or renumbers cards is not overlaid, since it goes through `insert-card` and `remove-card`.
The directive binds writing a finding's `fix`, not a Bouncer row's focus file.
The rubric points at `lyx loom validate-plan --help` for the overlay's format and never restates it (`PATTERN-producer-pointer-rule`).
Bound: per review file that proposes a target change, one baseline run and one combined run, plus one solo run per proposed fix when the combined run reports a new blocking finding, each slotted, since the verb validates through its wired index (Card 5).

The rubric also gains the enclosing-function directive: a finding whose proposed glyph stands for a comment or a line it cites names the function or method whose body encloses that line, confirmed with `lyx quarry resolve` that the line lies inside the glyph's span, never the member after or near it; when no member span holds the line, as for a free-standing comment between declarations, it names the file's self glyph.

`wordingClaims`' row for `loom-rubric-plan-review.md` gains claims pinning, in `## Proposed fixes`, the `validate-plan --overlay` run, the `{}` baseline, the solo re-run, the joint conflict and the `--help` pointer, and the enclosing-function directive with its `lyx quarry resolve` confirmation and its file self glyph fallback.

**ImpactSummary:** one stencil and its claims row; no Go code reads the new wording, and `TestStencilClaims` and the rubric marker test cover it.

**Commit:** 6: Plan-Review rubric validates proposed fixes with --overlay and names enclosing functions
