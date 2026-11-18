```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F8/F9 change set for agreement with its surroundings.

No findings.

- The marker lines match the repo's `//testtiming:keep <reason>` format, with the marker directly above each `func Test…` line.
- No other Go file in `internal/proc` repeats `/proc/self/cgroup` or `/sys/fs/cgroup`. The only other hit, `cmd/testtiming/resources.go:68`, is a separate package that has its own constant `cgroupRoot` and does not import `proc`.
- `tasklimit.go` has no build constraint. `export_linux_test.go` is Linux-only by its filename suffix, so the new aliases resolve.
- Both new aliases follow the existing doc-comment style in `export_linux_test.go`.
- The `go vet ./internal/proc` run was clean.

Tool calls:
1. grep for the two path literals across Go files: to find twin repetitions outside the changed hunks.
2. grep for existing `testtiming:keep` markers: to compare the marker format.
3. head of `tasklimit.go` and grep for `go:build` in both files: to confirm the constants are not build-constrained.
