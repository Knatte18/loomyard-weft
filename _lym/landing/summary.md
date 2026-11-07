# gitkit and hubforge initialisers write their fixture git config from Go

The fixture repository initialisers in `internal/gitkit` (`initRepo`, `initBareRemote`) and `internal/hubforge` (`initScratchRepo`, `initBareRepo`) no longer run one `git config` process per key after `git init`.
They call the new `gitkit.WriteFixtureGitConfig`, which appends the keys to the new repository's `config` file in one Go write.
The decision record puts the saving at about 5,600 git processes, about 2 s of wall time, per package run.

## What changes

- `gitkit.WriteFixtureGitConfig(gitDir, bare)` appends `core.fsmonitor = false`, `maintenance.auto = false` and `gc.auto = 0` to every repository's config.
  For a non-bare repository it also appends `user.name = Test` and `user.email = test@test.com`.
- The keys and values are the ones the initialisers set before, so the per-repository config layer stays.
  No fixture comes to depend on `gitkit.HermeticGitEnv` for these keys.
- The appended sections can repeat a header `git init` already wrote, such as `[core]`.
  Git reads a repeated section as one section, and none of these keys appears twice.

## Tests

- `TestCopiedRepoScenario`'s config step now reads every key from the copied repository and its bare origin with `git config --local --get`.
  It also checks that the bare repository carries no `user.name`.
- The new `TestInitRepos_WriteFixtureConfigKeys` in `internal/hubforge` runs each of its two initialisers on a fresh directory and reads back the keys it writes.

## Docs

The Layer A paragraph of `internal/gitkit`'s package doc names `WriteFixtureGitConfig` and the single write after `git init`.
