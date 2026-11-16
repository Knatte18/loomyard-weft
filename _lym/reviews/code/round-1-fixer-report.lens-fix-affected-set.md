```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The set under F1 is every doc line that says `lym gate testmap` works only from the prime. I searched markdown, Go and YAML files for `testmap` near "prime", "prime's HEAD" and "coverage map".

- The two PATTERN lines that were wrong (`pattern/PATTERN-test-speed.md` 221 and 292) are fixed.
- No other prime-only statement about testmap is left.
- The remaining hits are about other things: adopt/fabric "prime's head", and test helper names and comments that refer to the prime as the cwd the tests pass in.

I did not check F2 and F3 as sets, because neither premise is one.

Tool calls:
1. Bash grep of `.md`, `.go` and `.yaml` files for prime-only testmap phrasing. My context held only the two PATTERN lines the review named, not the rest of the repo.
