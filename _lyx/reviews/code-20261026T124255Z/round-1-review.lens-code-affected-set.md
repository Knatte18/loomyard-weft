```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked what each change forces outside the lines it touches, per change below.

**Per change: set found, covered or not**

- **Removed `in-place-edit` and `redirect-write` rows (`rows.go`)**
  - Their helpers and patterns: covered. No remaining reference to the removed names outside a stale comment (F2).
  - The two `Forms` entries in the no-denied-recovery scan: covered. `shellGuardForms()` reads the table, so they leave the scan list, and the self-check rows were updated.
  - The claudeengine doc, the `settings.go` header and the golden: covered, and `hookcmd_integration_test.go` moved to `machine-sleep` rows.
  - `docs/overview.md`'s shuttle row: covered. The third and fourth guardrails were renumbered, and the `python` steer's `sed -n` mention still holds.
  - `pattern/PATTERN-refusal-way-forward.md`: not covered (F1).
  - The `unquotedPrefixPattern` mention in the `commandPositionPattern` comment in `rows.go:61`: not covered (F2).
- **Rewritten `edit-directive.md`**
  - `editdirective/doc.go`, `loomcli/doc.go`, `loomcli/fmtverb.go`, `stencils.go` and `parentdirective_test.go` comments: covered.
  - `pattern/PATTERN-parent-directive.md`: covered.
  - Other stencils restating the old ban: none found. `webster-body-implementer.md:48` is the rename-tool step, which the board entry leaves out of scope.
  - Fixtures reading "Change files only with Edit or Write." (`websterengine/template_test.go:251` and `:289`): harmless and still true, since they stand in for the directive.
  - The `Rename`-card exception: gone from the directive. `rename_tools` still renders in the implementer body and is unchanged by the board entry.
  - `docs/research/.../report.md:358` quotes the old wording. It is a dated research report and not stale text.
- **`editRulePhrases` in `stencils.go`**
  - The unsplit "`sed -i`" literal would trip the denied-recovery scan, but `stencils.go` is outside the scanned surface (`contracts/stencils/` is scanned only as `.md` text and non-test Go under `internal/webster*`, `internal/shed*` and `internal/loom*`). Covered.
  - `replace_all` was dropped from the list. The directive no longer says it, so this is consistent.

**Findings**

- **F1: MEDIUM, consistency.** `pattern/PATTERN-refusal-way-forward.md:18` still lists `sed -i` and `perl -i` in the closed list of denied forms, "the deny set today".
  - The same file's line 23 treats `-i<suffix>`, `--in-place` and its abbreviations, and the short-cluster spellings as denied-but-pattern-shaped.
  - The decision record says those forms leave the scan with the row. The code no longer denies them, so the background file is stale.
  - Fix: drop `sed -i` and `perl -i` from line 18's list, and drop the in-place spellings clause from line 23.
  - Evidence: `pattern/PATTERN-refusal-way-forward.md:18` and `:23`; `cmd/lyx/norecoverydeny_test.go:28-33` (operator forms) and `:46`.
- **F2: LOW, consistency.** The comment above `commandPositionPattern` in `internal/shellguard/rows.go:61` says "as unquotedPrefixPattern reads them", but the diff deleted `unquotedPrefixPattern`.
  - Fix: reword the sentence to describe the shell-unit run directly, with no reference to the deleted symbol.

**Tool calls**
1. A repo-wide grep for the removed names, the old-ban wording and `rename_tools`. The manifest held none of these call sites.
2. A grep for edit-directive, `sed -n` and rename wording across stencils and Go. It looked for other stencils or prompts restating the ban.
3. `sed -n` on `webster-body-implementer.md` lines 40-56 plus greps of the webster stencils and the PATTERN files. This checked the rename step's wording and the PATTERN text.
4. `sed -n` on `PATTERN-refusal-way-forward.md` lines 12-28 and a grep of `deniedRecoveryForms`. The diff had not shown that file.
5. A grep on `operatorDeniedForms` in `norecoverydeny_test.go`, to confirm `sed -i` is no longer in the operator list.
