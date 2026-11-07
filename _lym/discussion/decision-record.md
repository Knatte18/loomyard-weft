## Goal

gitkit and hubforge initialisers write their per-repository git config from Go

Takes: gitkit's initRepo and initBareRemote, and hubforge's initScratchRepo and initBareRepo
Delivers: one Go write of today's keys and values into each new repository's config after git init, in place of one git config process per key
Exports: a gitkit helper that writes the keys, which hubforge calls

## Scope

The part changes these packages:

- internal/gitkit (about 90 changed lines)
- internal/hubforge (about 60 changed lines)

## Decisions

### D4 — Quiet git config written from Go

The `gitkit` and `hubforge` repository initialisers write their per-repository keys into the new repository's `config` file from Go in one write after `git init`, in place of one `git config` process per key.
The non-bare initialisers write `user.name`, `user.email`, `core.fsmonitor`, `maintenance.auto` and `gc.auto`;
the bare initialisers write `core.fsmonitor`, `maintenance.auto` and `gc.auto`, each as today.
The keys and values stay as today, so the per-repository layer stays and nothing comes to rest on `gitkit.HermeticGitEnv`.

Rationale: about 5,600 processes a package run, about 2 s of wall.
Keeping the layer removes the report's open question, whether a test runs without `gitkit.HermeticGitEnv`: the result does not depend on it.

## Constraints

- `PATTERN-leaf-packages`: the D1 reader keeps `internal/dotgit` standard-library only and spawning nothing; the background's list of what `dotgit` parses gains the worktree entries and refs.
- `PATTERN-gitrepo-client-boundary`: D1 is a local read, which the entry gives to `dotgit`; the porcelain fallback stays on `gitexec`.
- `PATTERN-fabric-boundary`: D1's change to `fabricengine.List` stays inside `internal/fabricengine`; `internal/websterengine` reads its own worktree's siblings through `dotgit`, as it reads them through git today, and through its present porcelain read on a reftable repository.
- `PATTERN-test-isolation`: every template is a hub `fabriccli.CloneAndWire` built plus the build function's steps; no hub is hand-assembled, and no test package wraps `hubforge` in its own fixture type.
  The background's hub-fixture paragraph gains the registered template.
- `PATTERN-test-speed`: each test keeps one `tiergate.Require` at the lowest tier that fits; D8 follows it.

## Auto-mode assumptions

- D1: a reftable repository falls back to the porcelain path.
- D3: the binding fold stays on git, for the reflogs, in place of the entry's "no `amend` and `push --force` per copy".
- D4: the per-repository config layer stays.
- D6: the survey's rows the report did not read again are out.
- D7: `newMergePairFixture` moves only for repeated layers, and the `internal/battencli` `beamHub` joins the moves.

## Open risks

## Acceptance criteria

- D4: a repository from each initialiser reads the keys that initialiser writes today, at today's values, through `git config --get`.
