```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `doc.go` line-14 change for consistency with its sibling statements.

No findings.

- `doc.go:14` now says the stencil "carries paths and, in the PATTERN directive, the PATTERN overview". This agrees with `doc.go:15`, with `spec.go:46`, and with the banner in `contracts/stencils/sampler/sampler-template-agent.md:8` ("bar the PATTERN overview inside pattern_directive").
- The repo-wide grep for "paths only" found no other claim about the sampler prompt. The `landingshed` hits (`prsession.go:104`, `doc.go:25`) describe a different session with no PATTERN directive, so they are not twins.

Calls made:
1. `grep -rn -i "paths only\|bar the PATTERN"` over `.go`, `.md` and `.yaml` files (excluding `_lym` and `.lym`): to find twin claims elsewhere, which my context lacked.
