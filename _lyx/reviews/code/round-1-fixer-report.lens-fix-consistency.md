```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (new `internal/reedengine/proctree_linux_test.go`, commit ae9a9c7e) for agreement with its surroundings.

no findings

Checked against what my context shows:
- **Tier tag:** `//go:build tier0` matches `proctree_test.go`.
- **Package and header:** `package reedengine` and the file header follow the sibling convention.
- **Key 17:** the test uses the literal `17` rather than `auxvClockTicksKey`. That is deliberate, since the review asks the test to pin the key against the kernel vector, and the literal also appears in `TestClockTicksFromAuxv` row `DeclaredRate`.
- **Build constraint:** the `_linux_test.go` suffix gives an implicit linux constraint, and the `golang.org/x/sys/unix` import is already used by `proctree_linux.go`.
- **`t.Parallel()`:** it comes first, as the review asked.
- **Twin left unchanged:** `proctree_test.go`'s header says the OS-suffixed seams are "compile-checked only, never run on this host". `proctree_linux.go`'s header says they are exercised for real on Linux. The disagreement predates this fix and the fix doesn't touch the sentence, so it is out of scope.

Tool calls made: none; my context already held every changed file.
