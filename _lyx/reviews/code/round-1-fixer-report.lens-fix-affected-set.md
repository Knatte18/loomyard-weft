```yaml
lens: fix-affected-set
outcome: reported
```

Check: F2, F3 and F4 searched; F1 callers checked.

**Findings**

- **LOW** — `pattern/PATTERN-test-speed.md:40-43, 49, 54, 56, 61, 70, 72, 103` (untouched by the fix, same file as the F2 edit). These lines still use the retired spellings `llm`, `tmux` and `integration` as tier or tag names. Line 42 is "`go vet -tags llm`", line 56 is "constrained to `llm`… `//go:build tmux || llm`", and line 70 is "constrained to the `tmux` tier… `integration || tmux` fails". The tier table at lines 10-14 names the tags as `tier0` to `tier3`, and the new line 19 says the scan refuses the old spellings. The review's F3 premise was "background file still describes the tag sets under retired spellings", and F2's premise was the same transition ending. The fix covered only line 19 of this file.
- **LOW** — `pattern/PATTERN-test-isolation.md:13-14, 24, 34`. These lines are clean. Every remaining hit is a tmux binary or socket-directory name, not a tag.
- **LOW** — `pattern/PATTERN-test-economy.md:24` says "a `tmux`-tier file". `pattern/PATTERN-testkit.md:15` says "Only `llm`-tagged test files import it". Both carry retired tag spellings, in files the review did not name.

I checked these only by grep. I did not verify whether the lines are stale or are exempt by the decision record's prose exemption. The rubric allows a finding only for text the fix reaches, so these are LOW at most.

- F4: no other site in `contracts/specs`, `docs` or `doc.go` lists the older spellings. The only other hit, at `loom-plan-spec.md:255`, is the word "spelling" in an unrelated sense.
- F1: `firstRetiredTierSpelling` has a single caller, `tierTagFailure` at `cmd/lyx/tiertags_test.go:112`. The `nil` case is handled because the type switch falls through to `return ""`.

**Unchecked:** the retired spellings in Go sources, comments and testdata outside `cmd/lyx` (the F1 and F4 sets beyond the review's locations). The guard test covers constraints, and the decision record exempts comments and prose.

**Tool calls**
1. One Bash grep chain over `pattern/`, `contracts/specs`, `docs`, `doc.go` and the Go sources. It checked the F2, F3, F4 and F1 sets, which my context lacked.
