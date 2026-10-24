# Card 1 — gate config keys for the dotnet scoper, filter, shared build, carry and inert paths

**Create:**
- `internal/gateslot#BuildStep`
- `internal/gateslot#buildFile`
- `internal/gateslot#decodeBuild`
- `internal/gateslot#Config.Inert`
- `internal/gateslot#Config.AllInert`
- `internal/gateslot#TestConfig_Inert`

**Edit:**
- `internal/gateslot#Tier`
- `internal/gateslot#tierFile`
- `internal/gateslot#decodeTier`
- `internal/gateslot#Config`
- `internal/gateslot#configFile`
- `internal/gateslot#decodeConfig`
- `internal/gateslot#ConfigOpenMaps`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#TestLoadConfig`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`

**Intent:**
Add every new `gate.yaml` key this task reads, loaded through `gateslot.LoadConfig` over `configengine` (Decisions 1, 4, 5, 6, 8; Constraint `PATTERN-config-layers`).

- `scoper` accepts `dotnet` beside `go` and `none`; the refusal for any other value says `want go, none or dotnet`, and `Config.Scoper`'s comment names `go`, `none` or `dotnet`.
- `Tier` gains three fields: `Filter string` (the tier's own `--filter` expression, key `filter`), `SharedBuild bool` (key `shared_build`) and `NoCarry bool`, set when the tier file holds `carry: false`.
  `NoCarry` is the negative form so a hand-built `Tier` in a test, whose zero value is today's behaviour, still carries; `tierFile` reads `carry` as `*bool`, so an absent key and `carry: true` both leave `NoCarry` false.
  `Tier.Command`'s comment names the new `{filter}` placeholder.
- `decodeTier` refuses a non-empty `filter` on a tier whose command lacks `{filter}`, at key `tiers.<tierN>.filter`, saying the command holds no `{filter}`; `LoadConfig`'s existing wrapper adds `lyx config gate` as the way forward.
- `Config` gains `Build BuildStep` (key `build`) and `InertPaths []string` (key `inert_paths`).
  `BuildStep` holds `About`, `Command`, `Dir` (default `.`), `Env` and `TimeoutMin`; a zero `BuildStep`, the template's `build: {}`, means no build step, and a caller tells a declared step by its non-empty `Command`.
  `configFile` reads `build` as `buildFile`, whose `TimeoutMin` is `*int`, and `decodeBuild` converts it.
- `decodeConfig` refuses, each at its key: a non-empty `build` without `command` (`build.command`), without `timeout_min` (`build.timeout_min`, missing), a `timeout_min` below 1 (`build.timeout_min`, `want at least 1`), and a tier with `shared_build: true` while `build` is empty (`tiers.<tierN>.shared_build`, saying the hub declares no build step).
  A non-empty `build` means any key set, so `build: {dir: x}` is refused for its missing command.
- `ConfigOpenMaps` returns `build` beside `tiers` and `tier_parallel`, so `configengine` carries its sub-keys whole through the layer stack and reconcile never prunes a hub's build.
  `ConfigSpec` classifies `build` as `CategoryExecutable` and `inert_paths` as `CategoryGuard`, both hub-file only; `configreg`'s `TestModules_EveryTemplateKeyClassified` then holds without an edit.
- `template.yaml` gains `build: {}` and `inert_paths: []`, each with a one-line comment naming its sub-keys or pattern rule, and its tier comment names `filter`, `shared_build` and `carry`; the scoper comment names `dotnet` as the C# type-name closure over test classes.
- `Config.Inert` matches one worktree-relative slash path against `InertPaths` through `path.Match`: a pattern without `/` against the path's base name, a pattern with `/` against the whole path; a pattern `path.Match` rejects matches nothing.
  `Config.AllInert` is true only when `InertPaths` is non-empty, `paths` is non-empty and every path is inert, so a hub with the empty list and an empty diff both behave as today.
- `doc.go` gains the keys, their categories, their refusals and the inert matching rule, and its refusal list's `a scoper other than go or none` and its tier-key list are rewritten to name `dotnet` and the new keys.

Tests:
`TestLoadConfig` gains rows: `scoper: dotnet` loads; a tier with `filter`, `shared_build: true` and `carry: false` beside a full `build` and an `inert_paths` list loads to the expected `Config` (`Build.Dir` defaulting to `.`); and each refusal above, with its exact message and the `fixIt` clause.
The template row keeps passing, since a zero `BuildStep` and an empty `InertPaths` compare equal under `equateEmpty`.
`TestConfig_Inert` is a table over `Inert` and `AllInert`: a base-name pattern (`*.md`) matching `docs/a.md`, a slash pattern (`docs/*`) matching `docs/a.md` and not `src/docs/a.md`, a non-matching path, a malformed pattern matching nothing, and `AllInert` false for an empty list, an empty path set and one non-inert path among inert ones.

**ImpactSummary:** gateslot only; the new fields are zero for every existing hub and test-built `Tier`, so `gatecli` and every `LoadConfig` caller behave as before until later cards read them.
