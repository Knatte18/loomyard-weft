```yaml
lens: code-tests
outcome: reported
```

Lens code-tests. List, per behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

- Step 2 scope list wording (`off` / `full` / `partial` guidance and the "Never skip" line): no test pins it. The diff changes only static stencil prose, which this lens exempts. The only test touching the stencil is `contracts/stencils/markers_test.go:62`, which pins the marker set (`baseline_path`, `capture_command`, `edit_directive`, `parent_directive`, `scope_note`, `subject_reading`). The diff leaves all six markers in place, so that row still holds.
- Step 4 baseline report wording (naming expected drifting tests and lines, naming skipped tests): no test pins it. Static prose, exempt.
- Code behaviour: the diff changes no Go code, no marker and no engine behaviour, so no behaviour exists that needs a test row.
- New or changed tests: the diff adds none. No test duplicates existing coverage, and no new assertion pins stencil prose.

**Findings**

None.

**Tool calls**

None. The manifest already held the diff, `markers_test.go` and the engine files.
