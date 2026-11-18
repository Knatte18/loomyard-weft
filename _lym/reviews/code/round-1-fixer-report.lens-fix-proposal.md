```yaml
lens: fix-proposal
outcome: reported
```

**F1 (`rows_test.go`, `//testtiming:keep` line above `TestRows_SteersOpenWithWholeCommandRefusal`)**

- LOW: the marker follows the doc comment directly. Sibling kept tests put a bare `//` line between the doc comment and the marker.
  Evidence: `internal/loomshed/discussionwrite_test.go:197-198` and `internal/burlercli/wiring_test.go:114-116`.
  Not a defect in function: the marker still sits directly above the `func Test…` line, which `pattern/PATTERN-test-speed.md:271` requires.
  It does break the file-level convention.
- I could not build a case where the reason text is wrong.

**F2 (`rows.go`, doc comments on `pythonRoutes`, `shellEditRefusal`, `headlessClaudeRefusal`)**

- LOW: each rewritten comment is a single line past the prose rule's 120-character limit, about 150, 140 and 145 characters.
  Splitting each at the colon would fix this.
- The reworded comments no longer claim the opening position, and the new phrase "the shared whole-command clause" names no symbol, so it breaks no rename.

Tool calls (2 of 6 used):
1. `grep` on `pattern/PATTERN-test-speed.md` for `testtiming:keep`, to confirm the placement rule and marker format.
2. `grep` on `internal` for existing markers, to compare the placement convention against sibling tests.
