```yaml
lens: code-affected-set
outcome: reported
```

**Affected set per change**

- **`CreateFileLink` (fslink):**
  - Covered: `fslink.go` header, `PATTERN.md`, `pattern/PATTERN-fs-links.md`, the `fslinkonly_test.go` allowlist reason, the Linux and Windows implementations, and `TestCreateFileLink`.
  - Nothing else in the repo documents `CreateDirLink` as the only primitive. The two remaining hits are under `docs/research`, which is not the subject.
  - There is no darwin file, but `CreateDirLink` has none either. Not a finding.
- **Codex aliases and engine (modelspec):**
  - Covered: `knownEngines`, `builtins()`, `template.yaml`, the `ConfigSpec` categories, `llm-model-spec.md`, and the tests in `load_test`, `registry_test`, `template_test` and `configsync_test`.
  - Not covered: the `internal/modelspec/template.go:14` comment, finding 1 below.
- **`codex` registered with module `codex.yaml`:**
  - Covered: `shuttleengines.go`, its `doc.go`, `shuttleengines_test` (names and scrub), `seam_enforcement_test` (allowed imports), `configreg_test`, and the `configstrictness` degrading set.
  - Reviewed, no change forced: `SessionNamers` (`shuttleengines.go:196`) looks up only `claude`, so codex gets no namer, which the diff never claims. `shuttleengine/config.go:83` is shuttle.yaml's own `claude` binary key. `keymove.go` holds claude-only migrations.
  - Not verified: whether `docs/overview.md` lists config modules elsewhere beyond the shuttle line the diff edited. Only one `claude.yaml` hit was inspected.
- **`CODEX_` scrub:**
  - Covered: `tmuxkit.ServerEnviron` and its doc, and `TestEnvScrub_MatchesTmuxkitServerEnviron`, which now includes `CODEX_HOME`.
  - `reedengine`'s tests use their own scrub fixture. Not affected.
  - `burlerengine/doc.go:315-317` names only claude keys as an example of what engines declare. It is still true.
- **`StartupChecker` (shuttleengine):**
  - Covered: the `engine.go` capability, the call in `run.go`, the `shuttleengine/doc.go` capability list, and `TestStartup_ChecksAfterStartup`.
- **Seam and enforcement scans:**
  - Reviewed, no change forced: `shuttleengine/seam_enforcement_test.go` and `orchengine/seam_enforcement_test.go` name `claudeengine` only as fixtures or a denied import. `interactivespawn_test.go` scans for `-p` fragments and codexengine has none.
  - Unverified: the `orchengine` denied-import scan names only `claudeengine`, so `codexengine` is not in its denied set. Not confirmed whether it matches by prefix, and not a diff change.
- **PATTERN declarations:** `PATTERN-fs-links` is the only declared entry the plan lists, and it lands in `PATTERN.md` with the declared text. The other declared entry, `PATTERN-engine-blocks`, belongs to a later part and is not in this scope.
- **Commit trailers:** none of the six commits carries a `Pattern-Departure:` trailer, so there is nothing to judge.

**Findings**

1. LOW, `consistency`: `internal/modelspec/template.go:14` still describes the seed template as `(sonnet/opus/haiku/fable with effort defaults)`. The template now also seeds `sol`, `astra` and `luna`. The diff updated the adjacent `load.go`, `registry.go` and `template_test.go` wording, but left this comment stale. Evidence: grep hit at `internal/modelspec/template.go:14`, against the new `template.yaml` entries.

No other member of the affected set is left wrong.

**Tool calls (5 of 6 used)**
1. `claudeengine` references in the seam and spawn scans. My context did not show how they treat engines.
2. The `fslink` directory listing and build tags, and `CreateDirLink` mentions in markdown. I needed to know whether other platforms or docs name the old primitive.
3. Files mentioning `claude.yaml`, to find lists of config modules.
4. Stale "four aliases" and "one registered engine" wording across the repo.
5. `CLAUDE_CODE_` and `CLAUDECODE` scrub copies outside the engine packages.

I also ran one more combined grep for `"claude"` in non-test Go code and the `claude.yaml` lines in `docs/overview.md`, so 6 calls in total.
