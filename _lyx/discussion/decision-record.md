# Decision record — scripted-edits-allowed

## Goal

Let every agent session lyx spawns, the orch included, edit files by script (`sed -i`, `perl -i`, `awk`, a shell heredoc or a redirect) where that is simpler than Edit or Write, so a mechanical change such as the `rename-binary-lyr` rename is done by script.

## Scope

- Remove the `in-place-edit` and `redirect-write` rows from `internal/shellguard`, with their patterns, steers, notices, the helpers only they use, and their test rows.
- Rewrite the `edit-directive` stencil so it permits scripted edits instead of banning them, and drop its `Rename`-card exception, which only existed to carve a hole in the ban.
- Update what describes the removed rows or the ban: the `claudeengine` doc and settings tests and golden, `docs/overview.md`'s shuttle row, the `editdirective` doc, `lyx loom fmt`'s doc, the stencils registry comments, the `PATTERN-parent-directive` background file, and the no-denied-recovery scan's self-check rows.
- Out of scope: the `python` row, the `raw-go`, `refuse-raw`, `computed-rm` and `machine-sleep` rows, and the rename-tool route a Webster implementer follows for a symbol `Rename`.

## Decisions

### Decision: remove both rows outright

**Decision:** Delete the `in-place-edit` and `redirect-write` rows from `Rows`, and delete `inPlaceEditCommandPattern`, `inPlaceCommandWordsPattern`, `redirectWriteCommandPattern`, `projectTargetPattern`, `notPrefixPattern`, `redirectTargetEndChars` and `unquotedPrefixPattern`, which no other row uses.
**Rationale:** The operator wants scripted edits allowed in every session; no config switch is asked for, so a switch would be an unrequested knob.
The removal lets through every `sed`/`perl` in-place edit and every shell redirect into a project file in every session, the orch included.
It is bounded by the rows that stay: `python` (on its switch), `raw-go`, `refuse-raw`, `computed-rm` and `machine-sleep` still deny as before, and Claude Code's own permission settings are untouched.

### Decision: the edit directive stays the one statement of the edit rule, now permissive

**Decision:** Keep the `edit-directive` stencil and its `edit_directive` marker in every opening stencil.
Its rule becomes: edit with Edit or Write, or by script with `sed`, `perl`, `awk` or a shell heredoc where that is simpler, such as a mechanical change across many sites.
It keeps the `lyx loom fmt` formatting route and the `.scratch/` rule for scratch files.
**Rationale:** The directive also carries the scratch-file and formatting rules, which still hold, and the opening-stencil marker scan and the no-restatement scan keep one place stating how files are edited.
Naming the shell tools rather than any script keeps the directive from steering an agent into the `python` row where its switch is on.

### Decision: no-restatement scan stays

**Decision:** `TestStencils_NoEditRuleRestatement` and `editRulePhrases` stay; only their wording changes from the no-script edit rule to the edit rule.
**Rationale:** The directive is still the only stencil that states how files are edited, and a stencil restating the old ban would contradict it.

### Decision: no PATTERN entry changes

**Decision:** The `PATTERN-parent-directive` overview line stays; its background file's clause names the edit rule instead of the no-script edit rule.
**Rationale:** The overview line names the edit directive, not the ban, so it stays true.

## Constraints

- `PATTERN-shell-guard-table`: the guard rows stay in `internal/shellguard`'s one table, and every engine renders every row.
- `PATTERN-parent-directive`: every opening stencil still renders the edit directive, and the `edit-directive` stencil is its only statement.
- `PATTERN-refusal-way-forward`: the no-denied-recovery scan reads the rows' `Forms`, so `sed -i` and `perl -i` leave its denied list with the row.
- The settings golden in `internal/shuttleengine/claudeengine/testdata` is pinned byte for byte and changes with the rows.

## Auto-mode assumptions

- The `python` row's steer, which suggests Edit or Write, stays as it is: it offers alternatives to Python and states no ban.
- The `.scratch/` rule for scratch files stays, since it guards against system temp directories, not against scripted edits.

## Open risks

- A session started before the change lands keeps the old rows and directive until it is relaunched.

## Acceptance criteria

- `shellguard.Rows` returns no `in-place-edit` or `redirect-write` row, and a `sed -i` or a redirect into a project file passes every remaining row.
- The Claude settings document and deny notice carry neither row's hook nor sentence, under every input combination the golden pins.
- The rendered edit directive permits scripted edits and no stencil restates the old ban.
- The no-denied-recovery scan no longer reports `sed -i` as a denied way forward.
