---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/fabricengine/doc.go:657
    summary: the fabric scratch-link section names a consumer module's text, the edit directive, though fabric is documented to know no module
    fix: In internal/fabricengine/doc.go, delete the sentence "The edit directive therefore has a project built there name the working tree's files by absolute path." at line 657, and keep the cost sentences at lines 654-656, which state the link's cost without naming a consumer.
    origin: handler
---

### [LOW] Fabric doc comment names the edit directive

**Location:** internal/fabricengine/doc.go:657
**Issue:** `pattern/PATTERN-fabric-boundary.md` (Fabric isolation) says fabric "serves every module the same way and knows none of them".
The new sentence at line 657 has fabric's package doc say what a consumer's stencil tells its agents to do.
The edit directive is not on the rule's literal list of IDEs, shuttle, the board and heddle, so this is advisory.
The same commit series moved fabric vocabulary out of the directive (`9fe6ce64`);
the reverse direction leaves fabric naming a consumer.
Lines 654-656 already state the cost in fabric's own terms, so the sentence adds nothing a reader of fabric needs.

## Verdict basis

The diff makes the change the amended entry asks for:
the link stays, and the edit directive documents it and warns against relative references under it.
The directive's claims match `fabricengine/doc.go:649`, where the link target is `<weft worktree>/<anchor>/.lym/scratch`, in the task's own pair.
"Your task alone" is therefore accurate, and it removes the apparent conflict with "Work only in your own worktree".
No other stencil, golden file or test asserts the directive's text:
`TestStencils_NoEditRuleRestatement` skips the `edit-directive` stencil, and the new sentences contain none of its phrases.
The summary's claims match the diff.

## Rejected

- Lens code-affected-set noted the directive warns agents and not card authors, and `loom-plan-spec.md:159` carries no warning: rejected, because every spawned role that runs a card receives the directive, which meets the entry's option.
- The commit message's cache-hit reason against `fabricengine/doc.go:651-653`: matches, so nothing to raise.
