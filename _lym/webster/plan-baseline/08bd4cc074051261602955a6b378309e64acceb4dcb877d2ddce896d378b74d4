# Card 15 — heddle-render

**Create:**
- `internal/lymdirs#HeddleDirName`
- `internal/heddleengine#Marker`
- `internal/heddleengine#DefaultBoltLockWait`
- `internal/heddleengine#RenderRequest`
- `internal/heddleengine#Report`
- `internal/heddleengine#Renderer`
- `internal/heddleengine#Renderer.Render`
- `internal/heddleengine#Renderer.read`
- `internal/heddleengine#fileText`
- `internal/heddleengine#Renderer.Show`
- `internal/heddleengine#Renderer.Passages`
- `internal/heddleengine#TestRender`
- `internal/heddleengine_test#TestRender_LeavesNoUntrackedFile`

**Extends:**
- `internal/lymdirs/doc.go`

**Uses:**
- `internal/heddleengine#Store`
- `internal/heddleengine#Store.Ordered`
- `internal/heddleengine#Decode`
- `internal/heddleengine#LookupTarget`
- `internal/fabricengine#BoardWriteLockFile`
- `internal/fabricengine#EventFor`
- `internal/fabricengine#ExcludeWayForward`
- `internal/lock#AcquireWriteLockWithin`
- `internal/gitkit#Git`

**Intent:** Render heddle's file targets into a worktree's root and read its prompt passages (D3), as the engine both fabric's seam and `lym heddle render` call.

`RenderRequest` is `Root` (the worktree's root), `ScratchDir` (the pair's never-tracked `.lym` directory), `BoltDir`, `Excluded` (fabric's exclude-list paths) and `Only` (file targets to render, all when empty); these are `fabricengine.WorktreeEvent`'s fields, handed over by the caller.
`Report` lists the files written and removed and the warnings.
`Renderer` holds `BoltLockWait`, zero meaning `DefaultBoltLockWait`, so a test shortens it.

`Render`:

- holds the per-worktree render lock, an `internal/lock` file `render.lock` under `<ScratchDir>/<lymdirs.HeddleDirName>/` (created on demand), for the whole read and write, so two renders of one worktree serialize and the last one's text stands whole;
- reads `heddle.json` through `read`, which holds the Bolt write lock, `<BoltDir>/<fabricengine.BoardWriteLockFile>`, only around the file read, acquired with `lock.AcquireWriteLockWithin(…, BoltLockWait)`, so it sees the old entries or the new, never an absent file in mid-move; a wait that runs out fails the render, writing nothing, with an error naming `lym heddle render` as the way to run it again (the lock order is the render lock, then the Bolt write lock);
- first checks the root's `.heddle`: a file, or a directory holding anything but `<target>.tmp` files, fails no render but writes no file target and adds a warning naming the path and the way forward, moving or removing it and then running `lym heddle render`;
- per file target of the table (only those in `Only` when given): builds the text with `fileText`; with no entry for the target, removes a file at the target path that opens with `Marker`, also when the path is off the exclude list, and writes nothing; with text, refuses the target when its file or `.heddle` is not in `Excluded`, adding a warning built by `fabricengine.ExcludeWayForward(path)`, and leaves a file there that lacks `Marker` untouched with a warning naming it (which covers a file the repository tracks);
- writes each file by writing `<Root>/.heddle/<file>.tmp`, overwriting a leftover from a crash, and renaming it onto the target, then removes the emptied `.heddle`, so a reader sees the old file or the new one whole and no file is left beside the targets.

`fileText` opens with `Marker`; for `claude-file`, when the root holds no `CLAUDE.md` there or in its `.claude` directory, it adds an import line for each `AGENTS.md` the root holds, `@AGENTS.md` and then `@.claude/AGENTS.md`; for `codex-file`, the repository's root `AGENTS.md` content when present; then the target's entries in `Ordered` order, each separated by a blank line.
It reports false when the target has no entry.
`Show` returns the text a target renders to now for a root, the prompt passages joined by blank lines for a prompt target, and writes nothing.
`Passages(boltDir, engine)` returns the `<engine>-prompt` entries' texts in order, read through `read`, and none for an engine with no prompt target.

The refusal text comes from `fabricengine.ExcludeWayForward` (card 9), so heddle's code never spells fabric's exclude-list key.
`lymdirs.HeddleDirName` is declared beside the other `.lym` subdirectory names, and `lymdirs/doc.go` lists it.

`TestRender` (tier0, in a new `internal/heddleengine/render_test.go` with `//go:build tier0`, temp directories as the root, scratch and Bolt directories, no git) rows:
a `claude-file` render joins its entries in order; a root `AGENTS.md` with no `CLAUDE.md` opens with `@AGENTS.md`; an `AGENTS.md` only in `.claude` opens with `@.claude/AGENTS.md`; a root `CLAUDE.md` gives no import, and so does a `CLAUDE.md` only in `.claude` beside a root `AGENTS.md`;
a `codex-file` render puts the root `AGENTS.md` first, and with no `codex-file` entry writes no `AGENTS.override.md`;
a marked stale render of a target with no entry is removed, also when its path is off `Excluded`;
an unmarked `CLAUDE.local.md` is neither overwritten nor removed and yields a warning naming it;
a target off `Excluded` is not written and the warning equals `ExcludeWayForward`'s text, and with `.heddle` alone off `Excluded` no target is written and the warning equals `ExcludeWayForward(".heddle")`;
a root `.heddle` file, and a `.heddle` directory holding another file, each yield a warning naming it and `lym heddle render` and no file written;
a leftover `.heddle/CLAUDE.local.md.tmp` is overwritten and the render leaves no `.heddle` behind;
a Bolt write lock held by the test past a 50 ms `BoltLockWait` fails the render naming `lym heddle render` and writes nothing;
with the test holding the Bolt write lock and `heddle.json` removed, a `Render` started in a goroutine stays blocked (no file written after 50 ms, with a `BoltLockWait` of seconds); the test then writes a new `heddle.json` and releases the lock, and the render ends with the new entries' text, so a read that skipped the lock would have found no store and rendered no file;
with the test holding `<ScratchDir>/heddle/render.lock`, a `Render` started in a goroutine writes nothing and leaves no `.heddle/*.tmp` while the lock is held, and completes once it is released, so two renders of one worktree never share a `<target>.tmp` and the last one's text stands whole;
`Only` naming `codex-file` leaves `CLAUDE.local.md` unwritten;
`Show` returns the text and writes nothing; `Passages` returns the `claude-prompt` entries in order and none for `codex` when it has none.

`TestRender_LeavesNoUntrackedFile` (tier1, in a new `internal/heddleengine/render_integration_test.go` with `//go:build tier1`) builds a `hubforge.CopyHub` hub, writes a `heddle.json` with a `claude-file` and a `codex-file` entry into its Bolt checkout, builds the prime's request from `fabricengine.EventFor`, renders, and asserts `git status --porcelain --untracked-files=all` in the prime is empty while both files exist.

**ImpactSummary:** New heddle rendering code; `lymdirs/doc.go` gains one line and `lymdirs` one declaration.
