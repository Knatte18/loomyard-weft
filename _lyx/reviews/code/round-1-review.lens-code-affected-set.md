```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — what the diff's changes force outside the lines they touch.

Per-change set (found, covered or not):
1. `in-place-edit` and `redirect-write` rows removed from `internal/shellguard/rows.go`. A repo-wide grep finds no remaining reference to `unquotedPrefixPattern`, `notPrefixPattern`, `projectTargetPattern`, `steerInPlace*` or `steerRedirect*`. The settings golden, `settings_test.go` and `hookcmd_integration_test.go` are in the diff. The rows' test rows were replaced by an "allowed everywhere" table. **Covered.**
2. Edit directive rewritten (`contracts/stencils/edit/edit-directive.md`).
   - **Test asserts:** the tests that assert on the directive text look for "Edit or Write". That string is still present in the new directive. These are `internal/editdirective/editdirective_test.go`, `internal/burlerengine/prompt_test.go`, `internal/frictionengine/reflect_test.go`, `internal/landingshed/describe_test.go`, `internal/loomengine/plan_test.go`, `internal/mergeresolve/mergeresolve_test.go`, `internal/shedadapters/bouncer_judge_test.go`, `internal/shedarbiter/prompt_test.go` and `internal/sheddriver/tacklerprompt_test.go`. **Covered.**
   - **`editRulePhrases`:** it dropped `replace_all` and the split `"`sed " + "-i`"`. The matching test rows in `parentdirective_test.go:214-238` are consistent with the new list. The edit-directive stencil is the only one that carries "Edit or Write", and no grep hit shows another stencil restating the rule. **Covered.**
   - **Webster implementer stencil:** `webster-body-implementer.md:48-50` still tells the implementer to run the rename tool. It never referred to the removed exception, and `loom-plan-spec.md` keeps its own "never a text or regex replace" rule. The wording stays coherent. **Covered.**
3. Docs and comments:
   - `claudeengine/doc.go`, `settings.go`, `docs/overview.md`, `docs/research/codex-feasibility.md`, `internal/editdirective/doc.go`, `internal/loomcli/doc.go`, `internal/loomcli/fmtverb.go`, the `stencils.go` comments, and `pattern/PATTERN-parent-directive.md` and `pattern/PATTERN-refusal-way-forward.md` are updated. **Covered.**
   - A grep for "in-place", "redirect", "no-script", "never with a script" and "rewriting tool" finds no leftover claim of a scripted-edit ban outside `docs/research/stencil-engine-audit-2026-10-24/report.md:358` and `docs/research/glyph-audit-2026-10-15/COMMANDS.md:5`. Both are dated audit records, not live docs, so I don't count them as findings.
4. No-denied-recovery scan (`cmd/lyx/norecoverydeny_test.go`). The `sed -i` and `perl -i` self-check rows were removed and one row now asserts `sed -i` is not denied. The `Forms` entries left with the rows, and `pattern/PATTERN-refusal-way-forward.md:18-23` matches. **Covered.**
5. `internal/loomcli/doc.go`. The task's edit only rewords the Format verb paragraph, and the parent's change to this file is not in this diff. No leftover claim of a ban remains in the file (grep of "edit"). **Covered.**

**Findings:** none.

Tool calls (3 of 6):
1. `git grep` for edit-directive and ban phrases across `contracts`, `plugins`, `docs`, `internal`, `cmd` and `pattern`. My context did not list the consumers of the "Edit or Write" string.
2. Read `webster-body-implementer.md` lines 40-56, and `git grep` for the removed helper symbols and for exception, shell guard and redirect wording in stencils and specs. I needed to check the rename-tool wording and the removed symbols.
3. Read `parentdirective_test.go` lines 205-245, and `git grep` for in-place and redirect wording in shuttle, shellguard and docs. I needed to see the edit-rule scan's test rows and any leftover row descriptions.
