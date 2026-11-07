```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change in `hub_test.go` through the fix-consistency lens.

**Findings:** no findings.

Points checked that did not become findings:
- **Twin in `gitkit_test.go`:** it pins the bare repository's missing `user.name` through `gitExit` and expects exit 1. The new `hub_test.go` check asserts the same exit 1 through `exec.Command`. The two agree on the contract.
- **Different spawn path:** `hubforge` is an external package and cannot reach the unexported `gitExit`. The `exec.Command` call is the only way to read the exit code there, so this is not a stale twin. The call skips `fixtureGitEnviron()`, but `--local` scope makes the process environment irrelevant to the result.
- **`user.email`:** the `hub_test.go` Bare row asserts it as well as `user.name`. The `gitkit` twin checks only `user.name`. This is stricter, not contradictory, and the review asked for both keys.
- **Scratch row:** it has no `absent` entry. It sets both identity keys, so none is needed.
- **Comments:** the test's doc comment says "Every key must sit in the new repository's own config". It does not mention the `absent` check, but the `absent` field has its own comment. I judge this not worth a finding.

Tool calls: 1. I read `internal/gitkit/gitkit_test.go` lines 100-125, because my context lacked the gitkit twin assertion for the bare repository's absent identity.
