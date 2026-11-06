```yaml
lens: code-affected-set
outcome: reported
```

Task restated: lens `code-affected-set`, what the diff forces outside the lines it touches.

**Per change, the set found:**
- **`volatile_files` / `volatile_keys` keys:**
  - Covered: `ConfigSpec` (guard categories and `HubOnlyWayForward`), `template.yaml`, `doc.go`, `ScopeRecord.Volatile`, and the writers `baseline.go`, `expect.go` and `verify.go` `recordDiffMode`.
  - Covered: readers `check.go` and `writeListing`.
  - Not changed: `internal/configreg/doc.go:23` says "such as drift's `diff` and `key_patterns`". That is an example, not wrong. `runsettings_test.go:271` refuses `key_patterns` only, which is the generic hub-only path.
  - No finding.
- **Judge guard and snapshot lifecycle (`TakeJudgeSnapshot`, `Changes`, halt texts):**
  - The only other `drift-judge` mentions are the role in `contracts/stencils/stencils.go:792`, `judge.go:18` and the `lymdirs` constant. None concerns the way-forward text. No stencil or spec tells anyone to delete the snapshot directory.
  - Covered: the `Changes` seams `TrailerCommit` and `FileAt` are added to `requireSeams`. I did not verify that every `Deps` fixture (`envkit`, `loomcli` `driftwiring`) fills them.
  - No finding.
- **`{skip_file}` migration and `CheckRetiredPlaceholders`:**
  - Covered: `PreflightDrift` flows through `shedrecipe.Env`, `entries_simple.go`, `preflightshed.Checks`, `loomcli/wiring.go` and the `envkit` `nilLegal` map.
  - Covered: the `preflightshed` `doc.go` is updated.
  - Not updated, only prose: `docs/research/drift/csharp-trial/...:18` still shows `--skip-file {skip_file}`. It is a dated trial record, not live documentation.
  - Gap found, see F1: `MigrateConfig` is a migration, not a reinterpretation.
- **`prefixedFailure` log line change:** the stencils failure's log message changed from `seat stencils check failed` to `a check failed`. The grep found no other reader of that string outside tests. The summary states it. Covered.

**Findings**

- **F1, LOW, decision.** `internal/driftengine/diffmode.go` (`MigrateConfig`, lines about 306–330), against `pattern/PATTERN-config-layers.md:3`.
  - The entry says "a meaning change ships a reinterpretation `hubreconcile` applies".
  - The diff removes `{skip_file}` through `MigrateConfig` and says in the summary that this is "a migration and not a reinterpretation".
  - The diff has no `Pattern-Departure:` trailer and no `PATTERN.md` or `pattern/` change. Both would be needed to excuse a difference from the entry.
  - The code's own rationale is plausible: a reinterpretation's marker would keep the reconcile from running at an operator's reconcile.
  - It is still an unrecorded departure from the entry's wording, because the placeholder's meaning did change. A reviewer must judge whether the migration path the entry names fits.
  - Evidence: the commit `6aecb9e9` message has no trailer; the `git diff --stat` over `PATTERN.md pattern` is empty.
- No other finding in this lens. No declared PATTERN changes were listed for this task, and none landed.

**Tool calls**
1. A grep and a ripgrep-style search for `key_patterns`, `drift-judge`, snapshot-directory text, `PreflightStencils` and `skip_file` across the repo. My context lacked the repo-wide dependants.
2. `git diff --stat` over `PATTERN.md` and `pattern/`, a grep of `PATTERN-config-layers.md`, a `Pattern-Departure` search of the commit messages, and a grep for `CheckRetiredPlaceholders` and `PreflightDrift` users. My context lacked the PATTERN diff status and the tests' use of the new symbols.
3. A grep of `configreg/doc.go`, `runsettings_test.go` and `PATTERN-config-layers.md` for hub-only key lists. My context lacked the wording.
