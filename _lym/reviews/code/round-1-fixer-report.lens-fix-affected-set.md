```yaml
lens: fix-affected-set
outcome: reported
```

Fix-affected-set check: the F1 premise is a set, every site that claims the sampler prompt carries "paths only". I reran that search once.

No findings.

- **Sites found:** two, `contracts/stencils/sampler/sampler-template-agent.md:8` and `internal/samplerengine/spec.go:46`.
- **Coverage:** both already end in "bar the PATTERN overview", and the fix changed `doc.go:14` to agree with them.
- **Stale claims:** none remain. The search matched only "paths only" within lines mentioning sampler or pattern.
- **Findings left unchecked:** none. The review has only F1.

Tool calls:
1. `grep -rniE "paths only"` over `.go`, `.md` and `.yaml` files, filtered to sampler or pattern lines: to find every site that repeats the claim F1 corrected, which my context did not list.
