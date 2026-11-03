# One base for a command's dir; a missing Drift dir fails by name
A command's `dir` now means the same directory in `gate.yaml` and `drift.yaml`: absolute as written, otherwise relative to the worktree root, and the worktree's anchored directory when unset.
Before, Drift joined a relative `dir` onto the anchored directory while the gate joined it onto the worktree root.
On a hub anchored at a subdirectory, one value worked in one file and broke the other.
The `cs` hub (anchor `lym-csharp/dotnet`) hit this: `dir: dotnet` ran the Drift capture in the nonexistent `…/dotnet/dotnet`.

The root is the shared base, so the gate's behaviour does not change; Drift moves to it.

## What changes

- **Drift resolves `dir` from the worktree root.**
  `driftengine` runs a step through `gateslot.ResolveCommandDir`, and `CaptureRequest` and `CompareRequest` carry the worktree root next to the anchor.
  `driftcli` and `loomcli` build their requests through shared helpers, so every capture and compare call site, including the side capture and the capture child, passes both directories.
- **A missing step dir fails by name before anything spawns.**
  The refusal names the resolved path, the `drift.yaml` key (`capture.dir` or `compare.dir`) and the way forward: `lym config drift --set <step>.dir=<dir>` from the prime, then `lym drift capture` and `lym loom resume` for the capture, `lym drift capture --side` for the side capture, `lym loom resume` for the compare.
  It replaces `fork/exec /usr/bin/sh: no such file or directory`, which Go reports for a failed `chdir` in the child.
- **A hub-wide write refuses a missing dir.**
  `lym config drift` and `lym config gate` writes, `--set` and editor save alike, check every command `dir` (`capture`, `compare`, tiers and `build`) against the prime through the new `configreg.Module.CheckDirs`.
  The refusal names the key and the resolved path.
  As with the existing load check, a write is refused only when the failure is new, so a file that already had a missing dir still takes an unrelated write.
- **Config templates state the base.**
  The `dir` comments in the drift and gate templates name the worktree root, the shared base and the unset default.
- **The notice to the orch keeps the full path.**
  `battencli` passes the worktree path, not the anchored directory, so a doubled or shortened directory is no longer hidden.
- **The baseline agent's prompt shows the real directory.**
  The capture command is described with the directory it runs in, relative to the worktree root.

## Carrying existing configs over

- A `gate.yaml` needs no change.
- A `drift.yaml` `dir` written relative to the anchor must be rewritten relative to the worktree root.
  For the `cs` hub, `dir: dotnet` now matches the gate and `dir: .` now means the worktree root; removing the key runs the step in the anchor, as `dir: .` did before.
- An absolute `dir` and an unset `dir` mean what they did.
- An unset Drift `dir` no longer loads as `"."`; `CommandStep.Dir` stays empty and the engine resolves it.

## Design decisions

- **The check sits at the write, not the load.**
  The board entry asked that the loader refuse a missing `dir`.
  `LoadConfig` still accepts it, because a load runs in every worktree and on every machine, where a dir may not exist yet, and refusing there would break unrelated commands.
  The write check catches the operator's typo, and the step's pre-spawn check catches the rest at run time with the resolved path.
- **The write check runs on hub-wide writes only.**
  A hub-wide write comes from the prime, so the dir is checked against it.
  A write to a worktree-layer file is checked by the existing load check alone.
- **The root as the base**, since the gate already used it and its docs and templates say so.

## To check by hand

- On a hub anchored at a subdirectory, run `lym config drift --set capture.dir=<missing>` and confirm the refusal names the resolved path, then run a capture with a deleted dir and confirm the Drift-Await stop and the notice carry the full path and the way forward.
- Confirm a real `drift.yaml` from an anchored hub, such as `cs`, still captures after its `dir` is rewritten as above.
