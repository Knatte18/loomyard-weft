```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Searches I reran, one per finding:
- **F1:** the only `memory.max` string literal left in non-test code is the declaration at `internal/proc/memorylimit.go:15`. `readMemoryLevel` and `scopeStartScript` use `cgroupMemoryMaxFile`. The other hits are comments, and `memorylimit_test.go:83` is test fixture data.
- **F2:** no `/dev/zero` or `/dev/urandom` hog is left in any Go test outside `_lym` and `.lym`. The hog in `tierrun_memorycap_linux_test.go` is now `head -c 268435456 /dev/zero | tail`. The remaining `sleep 300` hits in the proc tests and elsewhere are not memory hogs, and the proc ones are killed by the test or the cap.
- **F5:** the holder start exists in one place, `scopeStartScript`, and there it now runs under `setsid`. `doc.go:59` and `doc.go:65` never claim the holder shares the command's process group, so nothing there went stale.

Unchecked: F3, F4, F6 and F7 have no set premise.

Tool calls made:
1. Grep for `/dev/zero`, `/dev/urandom`, `yes` and `sleep 300` in Go files: F2's search.
2. Grep for `memory.max` and `read -r line` in non-test Go files in `internal`: F1's and F5's searches.
3. Grep for `holder` and `process group` in `internal/proc/doc.go`: the module doc was not in my context.
