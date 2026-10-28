```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer F1 and F2 proposals, as carried out by the fixer, for cases where they are wrong or break something.

No findings.

- F1: `git branch -v -l foo` and `-v --contains HEAD foo` still pass through their own list mode; `-v` and `-vv` pass through the no-operand rule; `-v -- foo` stays mutating; `--verbose=x` is handled; the `doc.go:303` wording matches.
- F2: `--format -x` and `--points-at -x` match git, a bare `git branch --` lists, and `git branch foo --` is mutating.

Tool calls made: none, because my context already held everything this lens needed.
