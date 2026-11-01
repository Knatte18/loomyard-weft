# Card 16 — heddle-worktree-seam

**Create:**
- `internal/heddleengine#Seam`
- `internal/heddleengine#Seam.Made`
- `internal/heddleengine#Seam.TookIn`
- `internal/heddleengine#requestOf`
- `internal/heddleengine#TestSeam`
- `internal/heddleengine_test#TestSeam_PullTakeIn`

**Uses:**
- `internal/heddleengine#Renderer`
- `internal/heddleengine#Renderer.Render`
- `internal/heddleengine#RenderRequest`
- `internal/heddleengine#Target`
- `internal/fabricengine#WorktreeEvent`
- `internal/fabricengine#WorktreeSeam`
- `internal/fabricengine#InstallWorktreeSeam`
- `internal/fabricengine#Fabric.Pull`
- `internal/fabricengine#Bolt.Pull`
- `internal/hubforge#CloneSecond`

**Intent:** Implement fabric's worktree seam in heddle (D6): render every file target when a worktree is made, and only the targets whose re-render paths a take-in changed.

- `Seam` holds a `Renderer` and satisfies `fabricengine.WorktreeSeam` (a compile-time assertion beside it).
- `Made` renders every file target into the event's root through `requestOf(ev, nil)`.
- `TookIn` selects the file targets whose `ReRender` paths include one of `ev.Paths`, and renders only those through `requestOf(ev, selected)`; with none selected it renders nothing and takes no lock.
- `requestOf` copies the event's root, scratch directory, Bolt checkout and exclude list into a `RenderRequest`.
- A render error is returned to fabric, which turns it into its warning (card 10); render warnings in the report are logged at Warn naming the file.
- A change to `heddle.json` itself is never a take-in, since fabric fires only for a worktree's code moves, so a Bolt pull re-renders nothing and the next take-in or a hand render takes `heddle.json` as it then stands.

`TestSeam` (tier0, in a new `internal/heddleengine/seam_test.go` with `//go:build tier0`, temp directories) rows: `Made` writes both file targets of a store holding a `claude-file` and a `codex-file` entry; `TookIn` with `["AGENTS.md"]` re-renders `AGENTS.override.md` (and `CLAUDE.local.md`, whose re-render paths hold `AGENTS.md` too) from a changed `AGENTS.md`; `TookIn` with `["src/x.go"]` writes nothing, a removed render staying absent; `TookIn` with `[".claude/CLAUDE.md"]` re-renders only `CLAUDE.local.md`.

`TestSeam_PullTakeIn` (tier1, in a new `internal/heddleengine/seam_integration_test.go` with `//go:build tier1`; not parallel, since it installs the process-wide seam) installs `Seam{}` with `fabricengine.InstallWorktreeSeam` on a `hubforge.CopyHub` hub whose Bolt checkout holds a `codex-file` entry, strips the three exclude lines from the prime's `info/exclude`, pushes a commit changing `AGENTS.md` from `hubforge.CloneSecond`'s hub, and pulls the prime with `Fabric.Pull`.
It asserts `AGENTS.override.md` opens with the marker and holds the new `AGENTS.md` first, the exclude lines are back, and `git status` in the prime shows no untracked file; then it pushes a `heddle.json` change to Bolt, pulls Bolt with `Bolt.Pull`, and asserts `AGENTS.override.md` is unchanged.
