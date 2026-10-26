```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against the repo's PATTERN rules (code-pattern-fit lens).

**Rules checked**
- `PATTERN-shell-guard-table`: held. The rows are removed from the one table in `rows.go`, and the leftover rows are untouched. `settings.go` and its doc comment name only the remaining rows.
- `PATTERN-parent-directive`: held. The `edit-directive` stencil stays the only statement, and `TestStencils_NoEditRuleRestatement` and `editRulePhrases` stay. The background file's clause is updated.
- `PATTERN-refusal-way-forward`: **broken**, see F1.
- `PATTERN-stencil-ownership`: held. The directive is read from the stencil store at call time, not embedded elsewhere.
- Declared PATTERN changes: none are declared, and no overview line changes. Held.
- CLAUDE.md "Docs land in the same commit": held. `docs/overview.md`, the `editdirective`, `claudeengine` and `loomcli` docs, and the PATTERN background file are all updated.
- CLAUDE.md semantic line breaks: held. The edit directive and the `.go` doc comments follow it.
- Commit message: held. There is no `Pattern-Departure` trailer, and none is needed because no entry forbids the answer key's way.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: held. The diff changes only table rows and assertions in existing tests, with no new tiers or time dependence.

**Findings**

F1. MEDIUM, class `consistency`. Location: `pattern/PATTERN-refusal-way-forward.md:18` and `:23`.
- The closed list of denied forms still names `sed -i` and `perl -i`, "the deny set today". Line 23 still describes the in-place spellings as pattern-shaped and outside the list.
- The diff removed the `in-place-edit` row, so its `Forms` entries `{"sed","-i"}` and `{"perl","-i"}` no longer feed the scan. The decision record's constraint says these forms "leave its denied list with the row".
- Neither the decision record nor the commit's file list covers this PATTERN background file, so the stale list stays.
- `cmd/lyx/norecoverydeny_test.go:228` now asserts that `sed -i` is not denied, which contradicts the background file.
- Fix: remove `sed -i` and `perl -i` from the line-18 list, and drop the in-place clause from line 23.

F2. LOW, class `consistency`. Location: `docs/research/codex-feasibility.md:14`.
- The deny-guardrails row still lists `sed -i` as a guardrail to port to Codex hooks. It is a research document, so this is minor, and the code-affected-set lens may cover it too.
- Fix: drop `sed -i` from that row.

Tool calls:
1. A grep across `*.md`, `*.go`, `*.yaml` and `*.sh` for stale ban and row-name strings, to find leftover descriptions the diff did not touch.
