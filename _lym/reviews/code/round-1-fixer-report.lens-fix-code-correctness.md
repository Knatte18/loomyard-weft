```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **`run.go` F2 branch:** an input of `sideways` gives `invalid --anchor "sideways"; want own-window` with no suffix. An input of `hidden` gives the same prefix plus the full reed clause. The empty, `overview` and `sideways` inputs all get the plain text. No other caller is affected, since the change is local to the closure.
- **`cli_test.go` rows:**
  - The assertion at `cli_test.go:635` is `strings.Contains`.
  - The `AnchorUnknown` row expects `invalid --anchor "sideways"; want own-window`, which the new code produces.
  - With the old code, the unchanged text would still contain that substring, so this row passes either way and cannot catch the misleading cause. The `AnchorHidden` row checks the full text, but no row asserts that a non-hidden value lacks the reed clause. The review asked only to make the `AnchorUnknown` row assert the plain text, which was done, so this is not a finding.
- **`wire_seams_test.go` F1:** the `proc` import is removed and `proc` had no other use there, per the grep. `fmt`, `os` and `filepath` are still used by `writeRun`, and `registry` is still used by the `MkdirAll` call. The expectation is now the constant `idle-stalled`/`no-output`. The parent's gate for both packages passed with 0 failures.

Tool calls:
1. `grep -n "wantErr" cli_test.go`: to confirm whether the comparison is exact or a substring match, which the shown context did not include.
