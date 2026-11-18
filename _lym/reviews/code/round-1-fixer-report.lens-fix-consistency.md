```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- No stale "opens every …" claim remains in `internal/shellguard/`. The only match is `rows.go:380`, `wholeCommandRefusal opens every steer`, which is now the one true statement.
- The three reworded comments (`pythonRoutes`, `shellEditRefusal`, `headlessClaudeRefusal`) say the same thing and agree with `wholeCommandRefusal`.
- The keep marker follows the form used elsewhere in the repo (`//testtiming:keep <reason>` directly above `func Test…`, as in `contracts/stencils/engineterms_test.go:256`).

Tool calls:
1. `grep -rn "opens every\|open every" internal/shellguard/`: checked for sibling comments still claiming the opening position, which my context did not hold for the rest of the package.
