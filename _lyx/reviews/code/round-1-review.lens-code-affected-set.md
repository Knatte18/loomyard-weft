```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's changes force outside the lines it touches (lens code-affected-set).

**Per-change set**

- **`runprofile.Load` return type (map → `Profiles`).** The non-test callers are `internal/boardcli/cli.go`, `internal/boardcli/runsettings.go`, `internal/configreg/runsettings.go` and `internal/battencli/runsettings.go`. All four are updated. The two test callers are updated too: `configcli_integration_test.go:559` and the `boardcli` tests. Covered.
- **`battenshed.RunSettings.Recipe` (new field).**
  - Producers: `internal/battencli/wire.go:1246` builds `RunSettings{Profile, Settings}` with no `Recipe`. This is intended, because `SeedChild` sets `Recipe` in `seamchild.go` before calling `PrepareRunSettings`. `internal/testkit/envkit/envkit.go:432` returns a zero `RunSettings`, which is harmless.
  - Consumers: only `battencli.prepareRunSettings`. Covered.
- **`Profiles.Lookup`.** `boardcli/runsettings.go:50` still checks only the profile name, as the decision record asks. Covered.
- **`lyx board profiles` envelope (new `defaults` field, nested settings).** The `cli.go` help text and example are updated. `boardcli/doc.go:66` is stale, so the set is not fully covered (finding below).
- **Profile document shape and the retired `budget`, `darn-wide`, `darn.review_*` and `merge_max_bounces` keys.**
  - A grep over `.go`, `.md`, `.yaml` and `.sh`, excluding `docs/research`, `_lyx` and `.lyx`, found no `darn-wide` or profile-`budget` reference.
  - `merge_max_bounces` remains only in migration and retire tests. Those assert the key is dropped, so they are legitimate.
  - The `review_max_bounces` hits in `configcli_test.go` are the `loom` module's own key, not a retired `darn.review_*` perch key.
  - The `profiles:`/`budget:` hits in `configengine`, `yamlengine` and `loomshed` are unrelated fixtures. Covered.
- **Docs and stencils.**
  - Updated: `runprofile/doc.go`, `battenshed/doc.go`, `boardengine/doc.go`, `configcli/doc.go`, `docs/overview.md`, `plugins/ly/skills/board/SKILL.md` and `orch-template-role.md`.
  - The stencil change matches the board entry. It no longer says `light` fixes review bounds, and it names the ladder profiles and the recipe default.
  - Missed: `boardcli/doc.go:66`.

**Findings**

1. **MEDIUM, consistency.** `internal/boardcli/doc.go:66` still documents the `profiles` verb's output as `{"profiles":[{"name":…,"settings":{…}}]}`. The diff added the `defaults` field in `cli.go:390` and changed `settings` to one block per recipe. The diff touched `boardcli/doc.go` only at line 55. The focus directive and CLAUDE.md both require the module doc to land in the same commit.
   - Fix: change line 66 to `{"ok":true,"defaults":{…},"profiles":[{"name":…,"settings":{recipe:{module:{…}}}}]}`, and state that `defaults` maps each recipe to its default profile.

2. **LOW, consistency.** `internal/boardcli/cli.go:487` describes the upsert `profile` field as `a "lyx board profiles" name, laid under run_settings at Seed-Child; "" clears it`. It does not say that an empty profile now takes the recipe's default, which `boardengine/doc.go` documents.
   - Fix: append "an empty profile takes the task's recipe's default".

**Tool calls (2)**
1. A combined grep for `runprofile` callers, retired names, `ReadRunSettings` implementers and `darn.review_*`. The manifest did not hold the callers outside the diff.
2. A grep of `profile` use in `boardcli`, `boardengine`, `configreg`, `battencli` and `configcli`. The manifest did not hold the `boardcli` doc and help text outside the diff.
