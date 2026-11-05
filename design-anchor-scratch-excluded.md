# The scratch dir lives at the anchor and is excluded there

`anchor-scratch-excluded` · Task · bug, fabric · running · Finalize

## Seen (2026-11-05)

In the C# sandbox hub `cs`, whose anchor is `lym-csharp/dotnet`, `git status` in the prime showed `?? dotnet/.scratch/`.
Fabric's scratch step links and excludes only the warp worktree's root `.scratch` (`/.scratch` in `.git/info/exclude`), but agents work from the anchor and write `.scratch/` there, so an anchored hub gets an untracked `<anchor>/.scratch`.

## Wanted (operator, 2026-11-05)

The rule: `.scratch` lives at the anchor, and the anchor's `.scratch` is excluded from git. Lym alone writes the exclude file: no hand edits of `.git/info/exclude`.

- Fabric's scratch step wires `<anchor>/.scratch` (the link into the weft-side `.lym` scratch) and writes `/<anchor>/.scratch` beside the other `/<anchor>/…` exclude lines; in a hub without an anchor, the anchor is the root, so nothing changes there.
- In an anchored hub, the root `.scratch` is not wired or excluded at all: nothing is to land at the repository root (operator, 2026-11-05).
- Reconcile moves existing anchored worktrees and the prime over: the root link and its exclude line go, the anchor's come in, so `cs` gets it without a hand edit.
