```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change the diff makes, the set it forces, and whether the diff covers it.

| Change | Set found | Covered |
|---|---|---|
| New `shell-edit` row (`rows.go`) | `settings_test.go` (installed under every input, PreToolUse count 4, `Prepare` counts 5 and 4), `settings-golden.json`, `claudeengine/doc.go`, `shellguard/doc.go`, `docs/overview.md` shuttle entry, `rows_integration_test.go` | Yes. |
| New `shell-edit` row, end to end | `hookcmd_integration_test.go:237-241` runs rows through the real hook command, with cases for machine-sleep and computed-rm | No `shell-edit` case. See F1. |
| `steerPythonDeny` and `noticePythonDeny` split | The only references are `rows.go:45,48,231,232`. `settings_test.go` reads steers through `shellRow(...).Steer`, not literals | Yes. The notice wording is relied on by nothing else. |
| `edit-directive.md` rewritten | Every stencil embeds it through the `{{.edit_directive}}` marker (e.g. `webster-template-master.md:23`). `TestStencils_NoEditRuleRestatement` (`parentdirective_test.go:140`) plus a grep for `edit or write`, `heredoc`, `never by shell` and `sed -n` under `contracts/stencils` find the phrases only in `edit-directive.md` | Yes. No other stencil restates the rule. The rewritten directive keeps the `lym loom fmt`, `.scratch/` and own-worktree rules. |
| `editRulePhrases` (`` "`sed -i`" `` changed to `"never by shell"`) | `parentdirective_test.go:217-228` synthetic rows, updated. `norecoverydeny_test.go:228`, updated | Yes. `heredoc` and `edit or write` still pin the directive. |
| `internal/editdirective/doc.go` | Describes the new rule | Yes. |
| Stale "scripted edits pass" statements | A repo-wide grep for `sed -i`, `perl -i`, "by script" finds none left. Only `docs/research/*` records (historical measurement reports, exempt) and `refscanner_test.go:175` (an unrelated ref scanner) remain | Yes. |
| PATTERN.md and `pattern/` | The range has no diff there. `PATTERN-shell-guard-table` needs no new text (the row goes in the one table). The rubric lists no declared PATTERN changes, so nothing is owed | Yes. |
| Commit message | A single commit. Its claims (new row, steer and notice, python steer, directive) match the diff. It carries no `Pattern-Departure:` trailer, and I found no entry the diff forbids | Yes. |

### Findings

**F1 — LOW — consistency**
- **Location:** `internal/shuttleengine/claudeengine/hookcmd_integration_test.go:237-241`.
- **Summary:** The end-to-end hook-command test has rows for machine-sleep and computed-rm but none for the new shell-edit row.
- **Evidence:** `shellEditPattern` is a long grep -E pattern with single quotes, a backtick and `[[:space:]]`, and it is rendered into the Bash hook command. Only `rows_integration_test.go` runs the pattern, and it does not go through the hook command. Quoting damage when the hook command is built would go unseen.
- **Fix:** Add a deny row such as `{"shell-edit", "sed -i s/a/b/ f", true}` and an allow row such as `{"shell-edit", "sed -n 1p f", false}` beside the machine-sleep rows.
- **Caveat:** This is the one gap in the affected set. If a test in another lens already drives every row through the hook command, drop this finding.

No other affected-set findings.

### Tool calls
1. Grep over go, md, yaml and json files for `sed -i`, `perl -i`, "by script", the python steer and notice constants, and the old steer sentence. My context lacked the full set of consumers of the old wording.
2. Grep for `editRulePhrases` and `editRuleSentences`, a list of the files that use the edit directive, and a `git diff --stat` of `PATTERN.md` and `pattern/`. My context lacked these call sites.
3. Read `parentdirective_test.go:135-165` plus a stencil grep for `edit or write`, `heredoc`, `never by shell` and `sed -n`. My context lacked the restatement guard and whether other stencils contain these phrases.
4. Grep for `computed-rm`, `machine-sleep` and `raw-go` across the repo, plus PreToolUse length assertions. My context lacked other consumers of the row names and counts.
5. Grep for the row names in `hookcmd_integration_test.go`. My context lacked that file, and it is the end-to-end consumer of the rows.
