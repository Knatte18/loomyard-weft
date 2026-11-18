```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing each change in the diff to the decision that asks for it.

**Decisions**
- **G1:** Find which config dirs the orch launch hands the shuttle config loader, and make it read the hub's layers. This is a board entry goal point.
- **G2:** Add a test showing that with `deny_python: false` at the default layer, an orch launch carries neither the notice nor the hook row. This is a board entry goal point.
- **A1:** Added after Discussion, "The python hook reads deny_python live". The hook reads the value on each call, and the notice follows the value at launch.
- **A2:** Added after Discussion, "Accept round 1 of Code-Review with its fixes". F2 is fixed by a standalone state-dir fallback inside the verb, with no change to hook commands or engine signatures.

**Per change**
- **C1:** The hidden verb `hook-python-allowed`: `internal/shuttlecli/hookpython.go`, its registration and pre-run skip in `internal/shuttlecli/cli.go`, and `shuttleengine.PythonAllowedHookVerb` in `internal/shuttleengine/engine.go`. Traces to A1.
- **C2:** Claude installs the python row always, announces it by launch value, and adds the live check (`internal/shuttleengine/claudeengine/settings.go`). Traces to A1.
- **C3:** Codex does the same (`internal/shuttleengine/codexengine/guards.go`, `codexengine.go`). Traces to A1, with engine parity.
- **C4:** The standalone fallback in `shuttleConfigDirs` plus `cliwire.StandaloneStateDir` (`internal/cliwire/standalone.go`), and `hookpython_standalone_test.go`. Traces to A2 (F2).
  - The F2 commits are 47d84ce3 and ac646ddc. Hook commands stay pure functions of the row, so neither the hook commands nor the engines' `Prepare` signatures change.
- **C5:** `internal/orchcli/denypython_integration_test.go`, the orch-launch test. Traces to G2 as amended by A1.
  - The test asserts that the notice is absent for `false` and that the row stays installed. A1 overrides G2's "nor the hook row".
- **C6:** Docs and comments: `docs/overview.md:683`, the doc.go files of `shellguard`, `claudeengine` and `codexengine`, the `rows.go` switch comment, the `config.go` field doc, `template.yaml`, and the `configcli.go` help line. Forced by C1–C3, which made the earlier statements stale.
- **C7:** Updated tests: claudeengine `settings_test.go`, `manifest_test.go` and the settings golden; codexengine `guards_test.go`, `manifest_test.go`, `prepare_test.go` and `usersetup_test.go`; and the hook-run tests in `hookcmd_integration_test.go` and `guards_run_test.go`. Forced by C1–C3.
- **C8:** The comment line-break commits (ccb9cf22, c3cde635, d30e1558, 0905e617). Forced by the gate's comment lint on C1–C7's comments.

**Per decision**
- **G1:** Built by investigation, not by a code change. `summary.md` says `orchcli.wireCLI` already loads `fabricengine.ConfigDirs(location)` (`internal/orchcli/cli.go:223`), and the context read confirms that. C5 exercises that wiring.
- **G2:** Built by C5, as amended by A1.
- **A1:** Built by C1, C2 and C3, with C6 and C7.
- **A2:** Built by C4.

**Findings:** none. Every change traces to a decision or is forced by a traced change, and every decision is built.

One stale line is outside my lens: `docs/overview.md:717` still says Codex installs "the python and publish rows following their switches". That belongs to code-affected-set.

Tool calls:
1. `git show --stat` of 47d84ce3, ac646ddc and f168372a, to confirm the F2 fix stayed inside the verb and `cliwire` as A2 requires. My context held the combined diff, not the per-commit split.
