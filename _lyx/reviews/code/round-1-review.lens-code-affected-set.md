```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change the diff makes, I looked for the callers, tests, docs, stencils and fixtures that read or assert it.

**Per-change list (all covered, no finding):**
1. **Long forms `attach`, `pause`, `resume` dropped from `shortcuts.yaml`.**
   - Set found: `shortcutcli/doc.go`, `shortcut_test.go` (`TestMount_File` now pins exactly `a`, `prime`, `p`, `r`), `cmd/lyx/shortcuts_test.go` (help and index assertions), `docs/overview.md` (module table, shortcuts and tender entries), tender help in `tendercli/cli.go`.
   - Result: all updated.
   - The remaining `attach` strings in `shortcut_test.go` (lines 114 to 226) come from its own `testShortcuts` fixture and malformed-YAML rows, not the shipped file, so they are correct.
   - D3 holds: tender help and `docs/overview.md` name `lyx p` and `lyx r`.
2. **`StartAliasCommand` removed.**
   - Set found: the registration in `cmd/lyx/main.go`, the "Available modules" help text, the `start` row in the sandbox coverage allowlist, `loomcli/doc.go`, the `newLoomCLI` comment, the `docs/overview.md` loom and bootstrap lines, the `README.md` lines, and the `clitree_test.go` guard (now rejects a bare `run` or `start` child).
   - Result: all updated.
   - The grep for `StartAliasCommand` and for `loomcli` in `main.go` found no leftover reference.
   - `loomcli` is still imported for `Command()`.
   - `orchcli/start_test.go:223` is orch's own `start` verb, unrelated.
3. **Messages renamed from `lyx pause|resume` to `lyx tender pause|resume`.**
   - Set found: refusals and notices in `adoptcli`, `fabricengine/originreconcile.go`, `hibernateengine` (`wake.go`, `doc.go`), `loomcli/arm.go`, `pauserecord/hold.go`, `pauserecord/record.go`, `tenderengine/resume.go` and `tendercli/cli.go`.
   - Also help text and doc comments in `loomcli`, `shedcli`, `reedcli` and `battencli`, plus `docs/skills.md` and `SANDBOX-CORE-SUITE.md`.
   - The diff also touches test files for these, which the diff stat shows and I did not re-open.
   - Result: updated.
4. **Acceptance criterion 4, repo-wide sweep.**
   - A grep over the whole repo, excluding `.git`, `_lyx` and `.lyx`, for `lyx (attach|pause|resume|start)\b` returned nothing, so no Go, doc, stencil, contract, script or sandbox file still names a removed form.
   - A second grep for `lyx`, a backtick or a quote followed by `attach|pause|resume|start` found only unrelated hits: arbiter `resume` verdicts in stencils, reed's own `attach` and `resume` verbs, and `lyx shed` and `lyx loom` verbs.
   - `contracts/stencils`, `plugins` and the command index have no leftover reference to a removed form.

**Findings:** none.

**Tool calls (3 of 6):**
1. Grep for `StartAliasCommand`, root `start` and alias leftovers, a wider grep for the removed command forms, and `git diff` of `cmd/lyx` and `shortcut_test.go`. My context had no repo-wide search beyond the first grep's empty result, and I needed the test diffs.
2. Grep of the shortcut and `cmd/lyx` tests for `attach|pause|resume`, grep of contracts, plugins, docs and tools for the bare short forms, and a `loomcli` grep in `main.go`. This confirmed the leftover `attach` fixtures are test-local and the import is still used.
