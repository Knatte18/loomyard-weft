# An anchored hub excludes the anchor .scratch from git

`anchor-scratch-excluded` · Task · bug, fabric · draft

## Seen (2026-11-05)

In the C# sandbox hub `cs`, whose anchor is `lym-csharp/dotnet`, `git status` in the prime showed `?? dotnet/.scratch/`.
Fabric's scratch step links and excludes only the warp worktree's root `.scratch` (`/.scratch` in `.git/info/exclude`), but agents work from the anchor and write `.scratch/` under their working directory, so an anchored hub gets an untracked `<anchor>/.scratch`.

## Wanted (operator, 2026-11-05)

`.scratch` is always excluded from git, wherever an agent writes it, and lym alone writes the exclude file: no hand edits of `.git/info/exclude`.

- In an anchored hub, the anchor's `.scratch` is treated as the root's is: wired as the same link into the weft-side `.lym` scratch, or at least excluded, and its exclude line written beside the other `/<anchor>/…` lines.
- Reconcile adds the line to existing worktrees and the prime, so `cs` gets it without a hand edit.
