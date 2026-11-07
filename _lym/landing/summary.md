# gitkit, hubforge: write fixture git config from Go after git init

The fixture repository initialisers spent one `git config` process per key, five for a work-tree repository and three for a bare one.
Across a package run that came to thousands of processes.
They now append all their keys to the new repository's `config` file in one Go write after `git init`.

## What changes

- `gitkit.WriteFixtureGitConfig(gitDir, bare)` is a new exported helper.
  It appends `core.fsmonitor=false`, `maintenance.auto=false` and `gc.auto=0` to `<gitDir>/config`.
  For a non-bare repository it also appends `user.name=Test` and `user.email=test@test.com`.
- `gitkit.initRepo` and `gitkit.initBareRemote` call it in place of their `git config` calls.
- `hubforge.initScratchRepo` and `hubforge.initBareRepo` call it too.

The keys and values are unchanged, so a repository reads the same through `git config --get` as before.

## Design decisions

- The per-repository config layer stays.
  Nothing comes to rest on `gitkit.HermeticGitEnv`, so a test's result does not depend on whether it runs under it.
- A bare repository gets no identity, as before: it takes no commits.
- The helper opens the file with `O_APPEND` and no `O_CREATE`, so it fails loudly when `git init` left no config.
  The initialisers panic on that error, as they do on a failed git call.
- The `gitkit` package doc describes the helper next to the quiet-config layer.

## Tests

- `TestCopiedRepoScenario` in `internal/gitkit` reads every key back with `git config --local --get` for the work-tree repository and the bare remote.
  It also checks that the bare remote has no `user.name`.
- `TestInitRepos_WriteFixtureConfigKeys` in `internal/hubforge` does the same for `initScratchRepo` and `initBareRepo`, including the absent identity on the bare one.

No tests were run as part of writing this description.
