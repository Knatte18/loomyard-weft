```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing per behaviour the test that pins it.

**Per behaviour, the pinning test:**
- Six new `PATTERN.md` entries, covering ID format, duplicates, entry line length ≤ 360, overview ≤ 16 KiB, and the headroom line. `internal/pattern/check_test.go` (the table at lines 42–70) and `headroom_test.go:45` pin these rules. A real-file check applies them to the repo's `PATTERN.md`, but I did not locate it. The overview is 8052 bytes and the longest line is 317 characters, so the entries pass.
- Content of the six entries (declared text, topic heading). No test pins it. The decision record rules out tripwire tests, and prose is exempt from test coverage.
- `(test)` marking on `friction-capture`. No test checks the marking. Friction tests exist (`internal/frictionengine/seam_enforcement_test.go`, `internal/websterengine/refusalnote_test.go`), so the marking is plausible.
- Package-doc citations of the restored IDs (`agentname`, `friction`, `hubreconcile`, `parentlock`, `reedengine`, `shedengine`). Comment text only, no behaviour, no test needed.
- The new `friction/doc.go` sentence, "`lyx loom start` vouches for the resume it spawns". This is a doc statement. The code behind it is unchanged, and `internal/loomcli/bootstrap_test.go` is the nearest existing coverage; I did not confirm it asserts the voucher.

**Test economy findings:** None. The diff adds, changes and removes no test files. Its only added behaviour is doc text, which is exempt from coverage.

**Tool calls:**
1. Grepped `internal/pattern/*_test.go` for `PATTERN.md`, and listed test files in the diff. My context lacked the pattern checker's test coverage and whether the diff touched tests; the diff touched none.
