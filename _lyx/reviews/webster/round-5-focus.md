---
round: 5
focus:
  - "In the review range, list every added or changed line in a Go comment, then check each for a comma, semicolon or colon followed by a clause with its own subject and verb, including the coordinating conjunctions so, and, but, or, yet and for. Settle whether any such line still holds two independent clauses, judged against PATTERN-comment-line-breaks and the semantic-line-break rule in CLAUDE.md. Scan the whole range by rule and do not stop at the first hit; a compound predicate that shares one subject and a quoted argument list are not violations."
  - "In the review range, list every added or changed line in a .md file, including docs/overview.md, pattern/PATTERN-completion-signal.md, contracts/specs/loom-status-spec.md and contracts/stencils/orch/orch-template-role.md. Settle whether any such line still joins independent clauses on one line, judged against the semantic-line-break rule in CLAUDE.md. Table cells and blockquotes stay on one line and are not violations."
---

The focus names the rule to apply across the whole range, because the same line-break rule has recurred with a different clause form each round.
