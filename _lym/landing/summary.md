# Burler fix: name the rerun reason, let forks push to scratch repos

A fix round that runs its fixer again now tells the fixer so, and the mutating-git guard lets a fork push inside a repository it created.

Closes #176
Closes #177

## Fixer rerun (#176)

Before each attempt, `BurlerFixProducer` in `internal/shedadapters/burlerfix.go` renames a leftover round fixer report to a timestamped name.
The fixer prompt never said why the step ran again, so the second session could not tell a finished round from an unfinished one.

- `burlerengine.RunOpts.Rerun` (a `FixRerun` with `Reason` and `Report`) makes `fixWork` append a paragraph to the fixer's work.
  It says the fix runs again, not for new work, why, and where the earlier report was archived.
  The fixer is told to check each finding against the target as it stands, keep the changes that hold, and write the report afresh.
- `archiveStaleOutput` in `archive.go` is split out of `archiveStaleOutputs` and returns the archived path.
- The reason is kept in the round snapshot (`fixer_rerun` in `round-<n>-snapshot.json`), keyed by the digest of the accepted review.
  A record under another digest is never told.
  A failure exit of the fix producer records its halt as the reason, even when it archived no report, and keeps the report an earlier call recorded.
  A retired round, archived whole or by the review producer, drops the record.
- A failed read or write of the record is logged and never fails the round; the next fixer is then told nothing of the rerun.

## Fork git in its own scratch repo (#177)

`internal/burlerengine/cluster.go` already let a fork run mutating git in a repository it created, but every `git push` was refused.

- A repository counts as the fork's own when the fork made it with `git init`, `git clone` or `git worktree add`.
- A `git push` passes when its destination leads into such a repository:
  a remote that `git remote add`, `git remote set-url` or `git clone` pointed into one, or a path into one.
  A push with no destination, or after a config key that may redirect it (`remote.`, `url.` or `branch.`), never passes.
  `--git-dir`, `--work-tree` and unresolvable directories never pass either.
- The hard-failure halt now says when the audit found no target change, instead of telling the fork to revert something that does not exist.
- `internal/burlerengine/doc.go` documents both rules.

## Design decisions

- The guard errs toward refusing: a remote it cannot resolve to a fork-created repository is treated as the target's.
- The rerun reason is stored beside the round's other snapshot data, so it survives a resume and is dropped with the round it answered.

## To check

Findings the code review left open, to check by hand against `internal/burlerengine/cluster.go` and `internal/shedadapters/burlerfix.go`:

- `git clone` of a remote whose name looks like a git repository sets the redirect flag.
- A config key passed in its attached `-c`/env form may bypass the redirect check.
- A failure exit that has no fixer report may record no reason.
- The push halt may claim the target repository is unchanged when it is not.
- Lexical ownership of a path admits symlinks and failed clones as fork-created repositories.
- The pre-attempt archive record has no test.
- A relative push path may be resolved from the wrong directory.
- Remote tracking may mark a push safe while git pushes elsewhere.

Not run by hand: a real fixer session through a rerun, and a fork pushing to a scratch repository.

## Merge danger

**Door:** two-way
**Blast radius:** burler
