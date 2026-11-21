```yaml
lens: code-scope
outcome: reported
```

Task: map each change in the diff to the board-entry goal point it builds, and each goal point to the changes that build it.

**Decisions.** The record's `## Decisions` section is empty and it has no `Added after Discussion` entries, so the decisions are the two "What to do" points under `## Goal`:
- **G1:** decide the commit rule for findings that share files, and state it in the burler fix prompt.
- **G2:** have the review's `location` and `fix` keys carry the seam a fix reaches, or let the review flag findings whose site is not named.

**Per change:**
- **C1** `internal/burlerengine/prompt.go#fixScopeRules`, the source-scope return string: adds the joint-commit rule for findings that share a changed file, with its `fix <id>, <id> —` subject format. It keeps the per-finding commit for the rest. → G1. This string renders into `burler-step-fix.md`'s `{{.fix_scope_rules}}`, which is the burler fix prompt.
- **C2** `internal/burlerengine/prompt.go#fixScopeRules` doc comment, from commit a938c6da, reflowed by 5a4dad30 and 93b4c0f9: restates what the function returns. → Forced by C1.
- **C3** `internal/burlerengine/doc.go` FixScopeSource bullet: the package doc's statement of the same commit rule. → Forced by C1.
- **C4** `contracts/stencils/burler/burler-review-format.md`, the `location` rule: `location` names the site the fix changes, and an unfound site gets a `fix` that opens `site not found —`. → G2. It takes both options the entry offers, carrying the seam and flagging an unnamed site.
- **C5** `contracts/stencils/burler/burler-step-fix.md:27-28`: tells the fixer to find the site before it edits when the marker is present. → G2, as the consumer side of the marker C4 adds.

**Per decision:**
- G1 → C1, with C2 and C3 forced by it.
- G2 → C4 and C5.

**Findings:** none. Every change traces to G1 or G2 or is an edit C1 forces, and both goal points are built. The entry's "Separately …" notes, on the judge-call timeout mismatch and parent-finding enforcement, sit outside "What to do", so leaving them out is no scope gap. The landing summary says so too.

Not this lens's, left to code-affected-set: whether the diff should also have updated `internal/burlerengine/verdict.go:66` ("Location points at the offending content") and the "commit-per-fix" wording at `contracts/stencils/burler/burler-step-fix.md:98`.

Tool calls: none; the inherited context held the diff, the record, the board entry and the commit log.
