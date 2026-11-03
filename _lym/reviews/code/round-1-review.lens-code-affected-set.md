```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set** — what the D9/D10 changes force outside the lines they touch, per change.

**Per-change set**

1. **Hook settings and the guard entries.** I searched for callers of `hookSettings` and `hookSetting`.
   - Production caller: `codexengine.go:145`, updated.
   - Tests: `command_test.go` and `prepare_test.go:93`, updated.
   - Covered.

2. **`shellguard.Row` gains `EngineTexts`, `SteerFor` and `NoticeFor`.** Every user of `shellguard.Rows` is accounted for: `claudeengine/settings.go`, its tests and helper, `codexengine/guards.go` and its tests, and `cmd/lym/norecoverydeny_test.go`.
   - Claude keeps reading `Row.Steer` and `Row.Notice`, so its render is unchanged.
   - `norecoverydeny_test.go:37` reads the rows through `shellguard.Rows`. It does not see the Codex texts, but they name no denied form, so nothing is wrong.
   - `PATTERN-leaf-packages` still holds for `shellguard`: the new imports are `maps` and `slices`, both standard library.
   - `rows_test.go:14` keeps the same import allowlist.
   - Covered.

3. **`DenyPublish` is no longer refused.** The `PublishGuard` row is gone from `prepare_test.go`, and `codexengine/doc.go` is updated.
   - No other code or doc says Codex refuses the publish guard.
   - Covered.

4. **Codex now has skill loading, shell guards and a deny notice in its prompt file.** One doc is left wrong, see the finding below.

5. **Rules and commits.**
   - `PATTERN-shell-guard-table` ("every engine rendering every row") is met, since `installedGuards` iterates the whole table.
   - `PATTERN-shuttle-provider-seam` is met: the changes sit under `codexengine`, and `shellguard` names Codex only as a registry-key string.
   - The directive's declared PATTERN changes list is empty, so there is nothing to land.
   - No commit carries a `Pattern-Departure:` trailer, and none is needed.
   - `shellguard/doc.go` is updated for the new fields and for the payload contract naming Codex.

**Finding**

- **MEDIUM · consistency · `docs/overview.md:423`**: the shuttle entry still says the `codex` engine "has no skill loading, shell guards, fork audit or creel hooks yet".
  - This diff adds skill loading and shell guards, so that sentence is now false.
  - The `codexengine/doc.go` update shows the repo's own docs were meant to land with the change.
  - The sentence should keep "no fork audit or creel hooks" and say Codex has skill loading and the shell guard rows.
  - The same line's guardrail prose, for example "steering the session to read a file with Read and change it with Edit or Write", describes the Claude render. It stays true for Claude, but it does not mention the Codex per-engine texts or `apply_patch`.

**Tool calls (4 of 6)**
1. Grep for stale statements about Codex guards and skills across `internal`, `contracts`, `docs` and the PATTERN files. This found the `docs/overview.md` line; my context did not hold any doc outside the diff.
2. Grep for callers of `hookSettings` and `hookSetting`, for `//lym:seam` markers on `shellguard`, and for every Go file using `shellguard.`. My context lacked the caller set.
3. Grep for docs, specs and config naming `codexengine` or `codex.yaml`, and for `Pattern-Departure` in the commit messages. My context held neither.
4. Grep `norecoverydeny_test.go` and `docs/overview.md` for `shellguard` and `Steer`/`Notice` use. I needed to know whether the scan reads the per-engine texts.
