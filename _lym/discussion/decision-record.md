## Goal

worktree list read in-process through internal/dotgit

Takes: dotgit's FindRoot and Read, fabricengine.List's porcelain parse, and websterengine's otherWorktrees
Delivers: a stdlib-only dotgit worktree reader matching git worktree list --porcelain, with a named refusal of a non-files ref storage; List and otherWorktrees read through it and fall back to porcelain on that refusal; the PATTERN-leaf-packages background lists the worktree entries and refs dotgit parses
Exports: the dotgit worktree reader, its entry type and its ref-storage error

## Scope

The part changes these packages:

- internal/dotgit (about 650 changed lines)
- internal/fabricengine (about 100 changed lines)
- internal/websterengine (about 30 changed lines)

## Decisions

### D1 — `worktree list` through a `dotgit` reader

`internal/dotgit` gains a reader of a repository's worktrees from the common dir: the main worktree first, then each `worktrees/<id>/` entry.
Per entry it reads `gitdir`, `HEAD` and `locked`, and resolves `HEAD` to a SHA through the loose ref, then `packed-refs`, in the common dir.
`fabricengine.List` and the `worktree list` call in `internal/websterengine/gitwrap.go` read through it and start no git process; `List` keeps its `WorktreeEntry` result and callers.
The reader takes any directory inside a worktree, as callers pass `AnchorPath()`, a subdirectory at anchor `backend`, and finds its repository by walking up as git's discovery does.
A directory that does not exist is an error with no walk-up,
so the fallbacks in `internal/fabricengine/junction.go` and `internal/fabricengine/scratchlink.go`, which call `List` on a pair path that may be gone, still fire.
The reader matches `git worktree list --porcelain` as git 2.53.0 prints it:

- The main worktree's path is resolved through symlinks, as git prints it; a linked worktree's path is the directory of the `.git` its `gitdir` names.
- Linked worktrees follow the main one sorted by path, compared case-folded where the repository's `core.ignorecase` is true, as git's path compare does.
- A relative `gitdir` (written under `worktree.useRelativePaths`, git 2.48 and later) resolves against its `worktrees/<id>/` directory; an absolute one is taken as is.
- An entry whose `gitdir` file is missing or unreadable is left out, as git leaves it out.
- An entry is prunable when it is not locked and its `gitdir` names a `.git` that does not exist; the file's age plays no part in `list`.
- An entry with no `HEAD` file, as one `git worktree add` is still writing, is listed detached with the all-zero SHA, as git lists it; `List` and its callers never fail on it.
- A detached `HEAD` gives `List`'s detached branch value; a branch whose ref does not exist yet gives the all-zero SHA.
- A bare main repository stays an error, as `List` returns today.

Bound: a repository whose `extensions.refStorage` is not `files` has no ref files to read, so the reader refuses it with a named error.
On that error both callers fall back to `git worktree list --porcelain`:
`List` to its present parse, and the `internal/websterengine/gitwrap.go` worktree read to its present scan of the `worktree` lines.
The porcelain path stays in both callers for that case alone.
The reader stays standard-library only and spawns nothing, as `PATTERN-leaf-packages` holds `internal/dotgit`.

Rationale: `worktree list` is about 15 percent of `internal/fabricengine`'s git processes and about 5 s of the package's wall, and `lym` calls `List` three or more times in each `Add` and `Remove`.
`internal/dotgit` is already the sole `.git` parser, and `PATTERN-gitrepo-client-boundary` gives local reads to it.
Evidence: git 2.53.0 on neve; `man git-worktree` (porcelain format, `list` ignores expiry); git 2.48.0 release notes (relative worktree links).
A scratch repository on neve, reached through a symlink, showed:

- The main path printed resolved.
- Linked worktrees in path order.
- A fresh worktree with its directory moved marked `prunable gitdir file points to non-existent location`, and a locked one not marked.
- An entry with its `gitdir` file removed left out.
- An entry with its `HEAD` file removed listed as `detached` with the all-zero SHA.

At the floor, git 2.43.0 (`internal/toolfloor`), `list` behaves as at 2.53.0 on every point above but the relative `gitdir`.
Sources: `builtin/worktree.c` and `worktree.c` at tags `v2.43.0` and `v2.53.0`, and the git 2.31.0 release notes.

- `list` keeps the main worktree first and sorts the linked ones with `pathsort`, an `fspathcmp` compare of the paths, in the same code at both tags.
- The porcelain `locked`, `locked <reason>` and `prunable <reason>` lines date from 2.31.0, whose notes say `list` "shows locked and prunable attributes in --porcelain mode".
  At both tags a locked entry is never prunable, and `list` passes no expiry, so an entry whose `.git` is missing is prunable whatever its age.
- `get_linked_worktree` returns no entry when `gitdir` is missing, unreadable or empty, and `get_worktrees` skips it, at both tags.
- With no `HEAD` file, `add_head_info` is the same code at both tags, so 2.43.0 lists the entry detached with the all-zero SHA as 2.53.0 does in the experiment above.
  A partial read of `refs.c` at both tags agrees: a missing ref resolves with the SHA cleared and no symbolic-ref flag.
- The main worktree's path goes through `strbuf_add_real_path` at both tags; a linked path is the `gitdir` content less `/.git`.
- 2.43.0 takes a `gitdir`'s content as written and never resolves a relative one, which only git 2.48 or later writes.
  The reader resolves it as 2.53.0 does, since a repository holding one was written by a git that reads it that way.
  The relative-`gitdir` parity test needs git 2.48 or later to write one, and skips below it, naming the version.

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

- D1: on a repository with a main worktree, a branch worktree, a detached one, a locked one with a reason, a locked one without, and a prunable one, the reader's entries equal what `git worktree list --porcelain` reports: path, HEAD SHA, branch or detached, and prunable, in the same order.
- D1: the same parity holds when the repository is reached through a symlinked path.
- D1: an entry with its `gitdir` file removed is left out, and an entry with its `HEAD` file removed is listed detached with the all-zero SHA, as git reports both.
- D1: a linked worktree whose `gitdir` holds a relative path resolves to the same path git reports, under git 2.48 or later; the test skips below it, naming the version.
- D1: a branch worktree whose branch ref sits only in `packed-refs` resolves to its SHA.
- D1: a repository with `extensions.refStorage` set to `reftable` gets `List`'s result and the `internal/websterengine` sibling read's result through the porcelain fallback.
- D1: `fabricengine.List` and the `internal/websterengine` sibling read start no git process on a files-backend repository.
- D1: `List` from a subdirectory of a worktree equals `List` from its root, and `List` on a directory that does not exist returns an error.

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.
