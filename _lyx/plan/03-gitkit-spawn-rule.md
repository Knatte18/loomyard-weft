# Card 3 — gitkit-spawn-rule

**Create:**
- `//cmd/lyx/gitkitspawn_test.go`
- `fabricengine/export_integration_test.go`
- `fabricengine/clone_hubscratch_integration_test.go`

**Edit:**
- `//cmd/lyx/tierpurity_test.go`
- `//cmd/lyx/hermeticenv_test.go`
- `fabricengine/export_test.go`
- `fabricengine/clone_test.go`
- `websterengine/config_test.go`
- `//CONSTRAINTS.md`

**Uses:**
- `gitkit/query.go`

**Intent:**
Replace the two hand-kept gitkit token lists in `cmd/lyx` with one rule (D4): every `gitkit` export except `gitkit.HermeticGitEnv` is a git spawn.

`cmd/lyx/gitkitspawn_test.go` holds the one shared definition: a function reporting whether file content references `gitkit.` followed by an exported identifier other than `HermeticGitEnv`, and which reference it found.
Matching on an exported identifier keeps a file-name mention such as `gitkit.go` (in `cmd/lyx/spawnobservability_test.go`) out of it.
The file names no gitkit export besides `HermeticGitEnv`, so it never trips either scan.

`tierpurity_test.go`'s `bannedTokens` drops `"gitkit.Copy"` and `TestTierPurity_UntaggedTestsSpawnNothing` also fails an untagged file the shared definition matches.
`hermeticenv_test.go`'s `gitSpawnTokens` drops `"gitkit.Copy"`, `"gitkit.MustRun"` and `"gitkit.SeedConfig"`, and `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain` also treats a package as git-spawning when any of its test files match the shared definition.
Every other token, both allowlists, the skip-dir set and the vacuous-scan floor stay verbatim (D9); the doc comments that justified the removed tokens are rewritten to state the rule instead of the list.

The untagged gitkit calls this newly flags move to tagged files, with no new allowlist entry:

- `fabricengine/export_test.go`'s git-spawning setup helpers (`newPlainWarpRepo`, `newPlainWeftRepo`, `newCommitFixture` and what only they use) move to `fabricengine/export_integration_test.go` (`//go:build integration`, `package fabricengine`); every caller is already integration-tagged.
- `fabricengine/clone_test.go`'s `initTinyRepo` and `TestCloneHub_CreatesHubScratchDir` move to `fabricengine/clone_hubscratch_integration_test.go`; the spawn-free tests stay untagged.
- `websterengine/config_test.go` only names `gitkit.SeedConfig` in a comment: reword it and keep the file untagged.

Re-run both scans over the tree and fix any further hit the same way (move the call to a tagged file, or reword a comment); a package newly flagged by the Hermetic Env scan gets a `testmain_test.go` calling `gitkit.HermeticGitEnv()`, never an allowlist entry.

`CONSTRAINTS.md`'s Test Tier Purity and Hermetic Git Test Environment sections state the rule (every gitkit export but `HermeticGitEnv` counts as a spawn, defined once in `cmd/lyx`) in place of token lists.

Prove the rule by planting, one at a time: an untagged `gitkit.RevParse` call (Tier Purity fails), a tagged `gitkit.GitStatusPorcelain` call in a package with no hermetic `TestMain` (Hermetic Env fails), and an untagged `TestMain` calling `gitkit.HermeticGitEnv()` (both pass).
Remove every plant.

What this tightens: untagged tests lose `gitkit.MustRun`, `SeedConfig` and `GitStatusPorcelain`, and the moved tests run under `-tags integration` only.
What it bans that is harmless today: `gitkit.RepoFixture`, which only the already-banned `CopyRepo` produces; the shared definition exempts a non-spawning export only by name.

**ImpactSummary:** Two cmd/lyx scans change their gitkit matching; three fabricengine/websterengine test files move or reword spawning code, with no test lost.
