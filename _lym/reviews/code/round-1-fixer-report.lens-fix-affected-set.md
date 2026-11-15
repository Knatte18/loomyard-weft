```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: the search for `Importers`, `BinaryBuilders`, `splitUnchanged` and `packageArgsOf` finds no code left. One test row name in `internal/impactset/targets_test.go:99` still says "the binary builder". It is a plain description, so I don't count it as stale.
- F2: the search finds no other coverage build start and no other place that drops or waits for an ahead build. `startBuild` has the one caller in `mapPackage`, and `takeBuild` and `close` are the only two places that discard a build. Both now cancel.
- F4: the search finds no other "own working directory" claim about the side-by-side runs.
- F3: I did not rerun a search. The finding names one test, and the fix covers it.

Tool calls:
1. Grep for the F1 symbols across `*.go` and `*.md`, to find leftover readers.
2. Grep for `.ahead`, `startBuild(` and `coverageBuild{` under `internal`, to find other build-drop sites. The `dropped` alternative in the pattern only added unrelated matches.
3. Grep for the "own working directory" wording, to find twin claims of the F4 doc text.
