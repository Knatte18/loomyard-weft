```yaml
lens: code-scope
outcome: reported
```

Lens: code-scope. I traced each change in the diff to the decision record's goal points and its one added entry.

**Decisions**
- G1 (Seen/Idea): find which config dirs the `lym orch` launch hands the shuttle config loader, and make it read the hub's layers.
  - No change. The summary says the launch already uses `fabricengine.ConfigDirs`, and `internal/orchcli/cli.go:223` confirms it. That makes this a finding, not a gap.
- G2 (Idea): a test where `deny_python: false` at the default layer gives an orch launch with neither the notice nor the hook row.
  - The added entry overrides the "hook row" half. The "notice" half still stands.
  - No change builds it. `internal/orchcli` and `internal/orchengine` have no test that mentions `deny_python`, `DenyPython`, the deny notice or `append-system-prompt`.
  - Notice absence is pinned only at engine level, through `buildDenyNotice`/`denyNotice` called with a cfg the test builds itself. The config path from hub layers to the launch is never exercised.
- A1 (Added after Discussion): the hook reads `deny_python` live on each call; the notice follows the value at launch. Built by C1–C4.

**Changes**
- C1: new verb `hook-python-allowed` (`internal/shuttlecli/hookpython.go`, its `cli.go` wiring, `PythonAllowedHookVerb` in `engine.go`). Traces to A1.
- C2: claudeengine always installs the python row with a live check, and splits installed from announced (`settings.go`). Traces to A1.
- C3: codexengine does the same (`guards.go`, `codexengine.go`). Traces to A1, with engine parity forcing the Codex half.
- C4: tests (`hookpython_test.go`, `hookcmd_integration_test.go`, `guards_run_test.go`, and the manifest, settings, guards, prepare and usersetup tests, plus the golden). They pin C1–C3 or follow changes C2/C3 force.
- C5: docs (`docs/overview.md`, the `shellguard`, `claudeengine` and `codexengine` `doc.go` files, the `config.go` `DenyPython` field doc, `template.yaml`). C2/C3 made them stale, so they are forced edits.
- C6: comment-reflow commits `d30e1558` and `0905e617`. These are forced by lint over lines the diff touches.

Every change traces to a decision or is forced by one. No unrequested scope.

**Findings**
1. **MEDIUM, scope.** `internal/orchcli` has no test for the board entry's asked-for check.
   - The Idea asks for an orch launch with `deny_python: false` at the default layer to carry no notice. That half survives the amendment, since the notice still follows the launch value.
   - No test drives the orch launch's config loading from a hub's default layer to the notice.
   - Evidence: `_lym/discussion/decision-record.md` `## Goal` Idea; `_lym/landing/summary.md` "The board entry's test is replaced"; no match in `internal/orchcli` or `internal/orchengine`.
   - Fix: add an orch-launch (or `orchcli` wiring) test where a hub has `deny_python: false` at the default layer and the prepared launch's notice has no python sentence.
2. **LOW, decision.** The writer dropped part of the board entry's Idea on their own call, outside the parent's added decision.
   - The summary calls the "neither notice nor hook row" test "replaced". The added entry overrides only the hook row; no recorded entry drops the notice half or the orch-launch scope of the test.
   - The summary treats this as the writer's design decision, not a parent correction, so it is not filed as an unrecorded parent correction.
   - Evidence: `_lym/landing/summary.md` "Design decisions"; the decision record's added entry.
   - Fix: covered by finding 1. Alternatively, the parent records the narrowed test with `lym loom decision add`.

Tool calls:
- grep of `internal/orchcli` and `internal/orchengine` for a `deny_python`/notice test: the manifest held only the `LoadConfig` call site in `orchcli`, not its tests.
