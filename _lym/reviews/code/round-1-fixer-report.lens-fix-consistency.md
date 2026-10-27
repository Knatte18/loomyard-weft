```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check the F1–F4 change set for twins and cross-references left stale. The lens asks for no list first.

**Findings**

1. **LOW**, F3's changed text is `internal/adoptcli/adopt.go:1` (`starts the adopt check`).
   - Summary: the same stale phrase "test suite" remains in sibling text that F3's reasoning covers.
   - Evidence:
     - `internal/adoptcli/adopt.go:98`, the `outcome` comment, still ends "the test suite started or the warning of a start that failed".
     - `internal/adoptcli/adopt_test.go:3`, the test file header, still ends "the beam steps and the test suite start".
     - `internal/adoptcli/deps.go:192` and `:195`, the `suiteLogName` and `suiteArgs` comments, still say "the adopt's test suite". `suiteArgs` itself says the command is "the hidden check, which runs the hub's adopt gate".
   - The review says the `deps.go` header and the `StartSuite` comment were already updated, so these lines were missed. The wording is now split between "adopt check" and "test suite".
   - The `deps.go:200` and `:203` mentions of "re-run the test suite" and "the test suite is not started under go test" are an error string and a go-test statement. They may be intentional.

2. **No finding on F1.**
   - The `heartbeat` and `logRunChanges` doc comments agree with the new `primeOwed` logic.
   - `dobbyengine/doc.go:56` ("a re-exec logs no change it did not see") still holds.
   - The existing `logRunChanges` tests, which call a tick on a nil memory, are unaffected because `primeOwed` is false there.

3. **No finding on F4.** The `Deps.Log` comment no longer names `Geometry.LogPath`, and no sibling comment repeats it.

4. **No finding on F2.** The `config.go` comment and the `doc.go` sentence state the same limitation in agreeing terms. The Open risks claim lives in the board entry, outside this repo.

**Tool calls**
- grep for "test suite" in `internal/adoptcli` and the docs, plus a grep for the heartbeat and prime phrases: to find twins of the changed text, which my context lacked.
- `git diff HEAD~4 -- internal/dobbyengine/passes.go`: to see the F1 change in its final form.
