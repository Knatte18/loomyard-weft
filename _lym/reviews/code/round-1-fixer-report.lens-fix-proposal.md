```yaml
lens: fix-proposal
outcome: reported
```

No findings.

The new line 14 of `internal/samplerengine/doc.go` reads "the `sampler-template-agent` stencil, which carries paths and, in the PATTERN directive, the PATTERN overview". I tried to build a case where that is wrong and could not.

- **Where the overview enters the prompt:** `spec.go:24-25,91-92` shows the stencil has an optional `pattern_directive` marker that `SessionSpec` fills from `pattern.Directive(..., RoleImplementer, ...)`. The overview therefore sits inside the stencil's rendered prompt. The new wording matches that.
- **Agreement with the stencil banner:** `contracts/stencils/sampler/sampler-template-agent.md:8` says "paths only, never file content, bar the PATTERN overview inside pattern_directive". The new doc.go line gives the same account.
- **Agreement with line 15:** the new text does not contradict "It renders ... the implementer PATTERN directive". The directive is the carrier of the overview, and line 15 is where it is rendered.
- **Surviving "paths only" claim:** the repo-wide grep for "paths only" found none left in the sampler package that conflicts with the change. The `landingshed` hits describe other sessions.
- **Wording:** "carries paths and, in the PATTERN directive, the PATTERN overview" is a little clumsy. It is still accurate and says what the reviewer asked for.

Calls made:
1. `grep -n -i "pattern" internal/samplerengine/spec.go`, `grep -n -i "pattern|paths only"` on the sampler stencil, and `git grep -n "paths only"`, in one Bash call. My context lacked the stencil text, the `spec.go` wiring of the marker, and any other copy of the old claim.
