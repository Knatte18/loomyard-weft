---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: contracts/stencils/loom/loom-template-plan.md:71
    summary: item 3 puts two sentences on one line, against one sentence per line
---

### [NIT] Two sentences on one line in "What a card is" item 3

**Location:** `contracts/stencils/loom/loom-template-plan.md:71`
**Issue:** The rewritten item 3 reads "… (`PATTERN-test-economy`). `verify:` commands are no substitute for test coverage;" on one line.
The repo's markdown rule is one sentence per line.
The old line had the same join, but the line is rewritten by this diff.
**Fix:** Break after "(`PATTERN-test-economy`)." so the `verify:` sentence starts its own line.

## Review notes

Range: no `_lyx/loom/rework/` round exists, so the range is `git merge-base main HEAD..HEAD` (`product.parent` is `main`).
It includes the Discussion commit's PATTERN edits, which the plan places outside the cards and which are not findings.

- Card 1: the removed phrases are gone from the spec, and the worked-example card bodies are untouched (only the intro line changed).
  `go test ./contracts/... ./internal/planparser/...` passes.
- Card 2: the new `wantAll` phrases ("Carries the coverage for the behavior it introduces", "no substitute for test coverage") are present in the stencil, and the `wantNone` group bars the old wording.
- Card 3: the claims match phrases in both rubrics, and each Test economy bullet points at `PATTERN-test-economy`.
  Each states a finding condition only, and each is bounded by naming the covering test.
- Card 4: the new lines match the claims, and `webster-body-verify-fix.md` is unchanged as the card says.
- No leftover old phrases anywhere in the repo outside the claims that bar them.
- Card groups carry no Rename or Delete targets, so no AST or `assert-no-callers` check applies.
