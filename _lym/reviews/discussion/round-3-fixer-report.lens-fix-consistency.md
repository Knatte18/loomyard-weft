```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text in the decision record still agrees with the decisions, criteria and constraints around it.

Two findings, both LOW.

1. **LOW: the review-file-format criterion doesn't name the new carried-class rule.**
   - Changed text: D3's carries bullet, which now says "A carried finding keeps its earlier finding's class". The round-check refusals and the D3 acceptance row were updated to match.
   - Problem: the "Review:" criterion still lists what the review-file format must state as "the carried finding's location-path rule", with nothing about class. A reviewer following the format would not learn that the class rule exists, while the round-check entry refuses a change of class.
   - Evidence: decision-record.md, Acceptance criteria, the bullet "- Review: `contracts/stencils/burler/burler-step-fix.md` tells the fixer to write the D1 frontmatter, and the review-file format states D3's `prior` and `carries` keys, the carried finding's location-path rule, …".

2. **LOW: D3 now relies on `PATTERN-cwd-resolution`, which the Constraints section doesn't list.**
   - Changed text: D3's validate-review paragraph, which says "resolves the worktree through `lymcwd` alone (`PATTERN-cwd-resolution`)".
   - Problem: the Constraints section lists each pattern the decisions bind to (fabric-boundary, sole-parsers, lym-files, tolerant-records, config-layers, stencil-ownership, refusal-way-forward). The new dependency is cited only inline in D3, so it is missing from that list.
   - Evidence: decision-record.md, `## Constraints`.

Further tool calls: none.
