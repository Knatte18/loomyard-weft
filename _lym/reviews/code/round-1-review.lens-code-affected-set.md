```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: for each output, key, message, file shape or behaviour the diff changes, check everything that reads or describes it, and report what the diff leaves wrong.

**Per change, the affected set and whether it is covered:**

1. **`ImportResult` gains `EmptyBody` and `LocalPaths`** (`internal/boardengine/intake.go:56-62`).
   - `board.go:155` re-reads `r.Entry` and keeps the other fields: covered.
   - `boardcli/intake.go:300` and `:318-323` read the new fields: covered.
   - `boardengine/doc.go:449-455` describes them: covered.
   - Existing `ImportIssue` callers at `boardengine/intake_test.go:173-290` are unaffected, because they build struct literals with no positional fields.
2. **`issueContent` changes** (`intake.go:200-216`): it adds import notes, and a whitespace-only body is now dropped instead of appended.
   - The fold tests' expected bodies at `boardengine/intake_test.go:180` and `:206` use the default "Issue body.", so they are unchanged.
   - `boardcli/intake_test.go:494` checks with Contains only.
   - No existing test uses an empty issue body: `render_test.go:146` sets `task.Body`, a different field.
   - Covered.
3. **`intake import` output gains `empty_body` and `local_paths`.**
   - `boardcli/doc.go:184-189`, the Long help at `boardcli/intake.go:94-95` and `SKILL.md:238-239` all describe the keys: covered.
   - `help_test.go:141` asserts only the slug-limit note: unaffected.
   - `contracts/orchrole/role-intake.md` covers the intake fork, which reads the output. It says nothing about the new keys. This is not forced, because the import note in the entry body carries the flag durably. No finding.
   - `docs/overview.md:471` describes intake at a level above the output shape, so nothing is stale.
4. **`selfreport create --attach`**, a new flag with new refusals.
   - The help Long, the package doc in `cli.go:9-10`, `docs/overview.md:621-622` and the friction stencil are all updated: covered.
   - `contracts/orchrole/orch-role.md:347` tells the orch to file lym bugs "with `lym selfreport create`" and gives no hint to carry run-directory evidence inline. That is one finding below.
5. **Friction reflection stencil wording** (`friction-template-reflection.md:57-62`).
   - It adds two rules: never file an empty body, and carry evidence inline.
   - It neither drops nor weakens an existing rule, and it restates the content rule as binding on attachments.
   - Consistent with the board entry's third bullet: covered.
6. **PATTERN fit.**
   - `boardengine` and `boardcli` now import `lymdirs`. Neither is a leaf package in `pattern/PATTERN-leaf-packages.md`.
   - `PATTERN-lym-files` wants the name to come only from `lymdirs`, and the code uses `lymdirs.DotLymDirName`.
   - `PATTERN-code-conventions`' named-constants rule is met in the code, including the boardcli help text.
   - The `.lym/` literals in `SKILL.md` and `doc.go` are prose.
   - This directive declares no PATTERN changes.
7. **Commit messages** (`e0b0cbca`, `8bc49289`, `92444823`, `790f3bda`).
   - Each claim is borne out by the diff.
   - There are no `Pattern-Departure:` or `Loosened-Check:` trailers, and no loosened assertions: changed existing tests only add fields and rows.

**Findings:**

- **LOW, consistency** — `contracts/orchrole/orch-role.md:347`. The orch role's own instruction to file lym bugs with `lym selfreport create` was not given the hint to carry run-directory evidence inline with `--attach` or quoted lines. Only the friction reflection stencil got that hint, so issues the orch files directly can still name only `.lym/` paths. Evidence: `contracts/orchrole/orch-role.md:347` against `contracts/stencils/friction/friction-template-reflection.md:57-62`.

Tool calls:
1. Grep for `intake import`, `ImportIssue` and `ImportResult` consumers and the orch-role selfreport line: the manifest had not searched for consumers outside the changed files.
2. Grep for empty-body test fixtures and fold-body assertions, and a read of `pattern/PATTERN-code-conventions.md`: the manifest lacked the existing tests and that background file.
3. Read of `board.go:145-165`, `role-intake.md`, `help_test.go:135-150` and `orch-role.md:340-352`: these are one-hop readers of `ImportResult` and of the intake and selfreport outputs, not in the manifest.
4. Grep for intake mentions in `docs/`: to check whether a user doc describes the import output shape.
