```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens over the F1 fix (one new retry test in `keeper_test.go`).

F1 is a single missing behavior, not a set of callers or sites. I reran its search for `.Retire(` in test files outside `keeper_test.go`, and it found no other tests. The new subtest therefore does not duplicate another test. Every other `Retire` case is a subtest of `TestChannelKeeper`, which the fixer saw whole.

The failed-drop-after-archive path has one site, `retire` at `keeper.go:443`. The new subtest exercises it by corrupting the forwarding file during `Archive`, and `refused[orch]` is set there. The retry needs `retryRefused`, which the subtest passes.

Findings: no findings.

Unchecked findings: none.

Tool calls:
1. `grep -rn "\.Retire(" internal --include=*_test.go`, excluding `keeper_test.go`, to see whether any other test already covers the retire retry. My context held only `keeper_test.go`.
