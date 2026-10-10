# Card 2 — buildinfo-production-channel

**Edit:**
- `internal/buildinfo#Channel`
- `internal/buildinfo#IsDev`
- `internal/buildinfo/buildinfo_test.go`
- `internal/buildinfo/doc.go`
- `tools/deploy#run`
- `tools/deploy/main_test.go`

**Create:**
- `plan:internal/buildinfo#ChannelProduction` -> `const ChannelProduction = "production"`
- `plan:internal/buildinfo#ChannelDev` -> `const ChannelDev = "dev"`
- `plan:internal/buildinfo#IsProduction` -> `func IsProduction() bool`

**Prosa:**
- `//pattern/PATTERN-leaf-packages.md`

**Intent:** D2: `buildinfo` gains a production channel beside `dev`.
`ChannelProduction` and `ChannelDev` are the two stamp values, `IsDev` compares against `ChannelDev`, and `IsProduction` is the exact-match accessor for `ChannelProduction`; an empty `Channel` is neither, and `Channel`'s doc comment stops calling empty "production": empty now means unstamped.
The package stays import-free; the leaf-packages background's `buildinfo` section names the two accessors instead of `IsDev()` alone.
`tools/deploy`'s `run` stamps `-X github.com/Knatte18/loomyard/internal/buildinfo.Channel=production` on the production route, the one `update-plugins.sh` and `update-plugins.cmd` reach, and keeps the `dev` stamp on `-dev`; `TestLdflagsPath_MatchesBuildinfoChannel` asserts both ldflag strings are present in `main.go`.
The existing `buildinfo` table test becomes a table over both accessors, so a value is asserted for `IsDev` and `IsProduction` on every row (`production`, `dev`, empty, case and whitespace variants).

**ImpactSummary:** `IsDev` keeps its behavior for every caller; the only observable change is a production deploy's binary now reporting `IsProduction()` true, which nothing reads until card 3.
