---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/samplerengine/doc.go:14
    summary: The package doc still says the sampler stencil carries paths only, which the new implementer PATTERN directive contradicts two lines below it.
    fix: In internal/samplerengine/doc.go line 14, change "which carries paths only:" to "which carries paths and, in the PATTERN directive, the PATTERN overview:" so that line agrees with line 15 and with the SessionSpec doc comment and the stencil banner, which already say "paths only, never file content, bar the PATTERN overview".
    origin: handler
---

### [LOW] Package doc still claims the sampler prompt carries paths only

**Location:** internal/samplerengine/doc.go:14

**Issue:** The diff updated doc.go line 15 to say `SessionSpec` renders the implementer PATTERN directive from the task worktree's PATTERN overview.
Line 14 of the same paragraph still says the `sampler-template-agent` stencil "carries paths only".
The prompt now inlines the PATTERN overview, so the two lines disagree.
The `SessionSpec` doc comment (`spec.go:46`) and the stencil banner (`sampler-template-agent.md:8`) were both reworded to "bar the PATTERN overview", which leaves doc.go as the one place the old claim survives.

## Rejected

- Fork `code-tests`, LOW `design`: no test makes `pattern.Directive` fail inside `SessionSpec`, so the wrap at `spec.go:81-84` is unexercised. Rejected: the branch is a pass-through of an operational error that `internal/pattern`'s own tests pin, and reaching it from here needs a corrupted stencils directory, so a row would restate `pattern.Directive`'s behaviour and not the sampler's; `DarnTable` and `VerifyTable` carry the same unexercised branch.
