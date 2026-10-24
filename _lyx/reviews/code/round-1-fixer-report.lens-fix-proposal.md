```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F5, all fixed as proposed.

no findings

- F1: the documented envelope matches the `lyx board profiles` sample at `internal/boardcli/cli.go:376` and the `defaults`/`profiles` keys built at `cli.go:390`. I found no case where it is wrong.
- F2: help text says an empty profile takes the recipe's default. That agrees with the "no profile takes the recipe's default profile" row in `runsettings_test.go`.
- F3: `TestTemplateDocsProfile` asserts `docs`' darn block and the missing loom block. It would fail if the template's writer or Code rounds changed. The tier-0 run passed, so the `int` types in the `DeepEqual` match what `Load` yields.
- F4: the new row fails if prepare skips loading for a profile-less task, because nothing would reach Stuck. The tier-0 run passed, so the `light: 3` file does produce the `profile "light"` refusal from `Load` for it.
- F5: the new comment agrees with the existing TestLoad row "a profile named defaults is refused". That row expects `recipe "loom" must name a profile`, so the entry is read only as the defaults mapping.

Tool calls: none; the context already held every changed file and the test run.
