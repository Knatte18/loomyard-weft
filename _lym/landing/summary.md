# One base for a command's dir, and a missing Drift dir fails by name

On a hub anchored at a subdirectory, the same `dir` value pointed at two different directories.
`gate.yaml` joined a tier's or build's relative `dir` onto the worktree root.
`drift.yaml` joined a capture's or compare's `dir` onto the anchored directory.
A Drift step with a missing `dir` failed as `fork/exec /usr/bin/sh: no such file or directory`, and its way forward only repeated the failing run.

## What changes

- **One base.**
  `gateslot.ResolveCommandDir` holds the rule for every hub command's `dir`: an empty `dir` runs in the worktree's anchor, an absolute one is used as written, and a relative one is joined onto the worktree root.
  This is the rule the gate already used.
  Drift now follows it too: `CaptureRequest` and `CompareRequest` carry `WorktreeDir` next to `AnchorDir`, and `driftcli` and `loomcli` fill it.
  `LoadConfig` no longer fills in `"."` as a step's default `dir`, so an unset `dir` stays empty and means the anchor.
- **A missing Drift dir fails by name.**
  `runStep` checks the resolved `dir` before it waits for a gate slot.
  A missing `dir` fails with the resolved path, the `drift.yaml` key (`capture.dir` or `compare.dir`), and the way forward:
  `lym config drift --set <step>.dir=<dir>` from the prime, then `lym drift capture` and `lym loom resume` for a capture, or `lym loom resume` for a compare.
  A failed capture puts this text in its record's reason, so the Drift-Await stop carries it.
- **Config writes refuse a missing dir.**
  `configreg.Module` has a new `CheckDirs` hook, which `drift` and `gate` set: `driftengine.CheckDirs` and `gateslot.CheckDirs`.
  A hub-wide `lym config <module> --set` and a hub-wide editor save run that check against the prime's worktree, next to `CheckLoad`.
  A `dir` that is not an existing directory there is refused with its key and resolved path.
  The file is put back as it was.
  As with `CheckLoad`, a write is refused only for a failure that the file did not already have.
- **Templates name the base.**
  Next to the `dir` key, each template now says that `dir` is absolute or relative to the worktree root, that no `dir` means the anchor, and that a hub write refuses a `dir` missing in the prime.
- **Notices keep the full path.**
  The batten and dobby run notices now shorten paths against the task's worktree root instead of its anchor.
  A doubled `dotnet/dotnet` now reads as such, not as `dotnet`.
  The cost on a subdirectory-anchored hub is a slightly longer report path, such as `dotnet/_lym/...`.

## Design decisions

- The shared base is the worktree root, not the anchor.
  The gate already resolves against the root throughout `gatecli`: tier runs, module discovery and the empty-tier check.
  The gate is also in wider use than Drift.
  Moving Drift to the root changes one small resolver, while moving the gate would change many.
- **Carry-over.** On a hub anchored at the worktree root, nothing changes.
  On a hub anchored at a subdirectory, an explicit drift `dir` written against the anchor needs rewriting.
  The `cs` hub's `drift.yaml` sets `dir: .`, which until now meant the anchor `dotnet`.
  It now means the worktree root.
  The fix is `lym config drift --set capture.dir=dotnet`, the value its `gate.yaml` already uses, or an empty `dir`, which means the anchor.
  Do the same for `compare.dir` when it is set.
  No migration rewrites the value automatically: the right rewrite depends on each hub's anchor, and the config migration hooks never see it.
- "The config loader refuses a missing dir" is read as the check on hub-wide `lym config` writes and editor saves (`CheckDirs`), not as part of `LoadConfig` itself.
  `LoadConfig` runs in every worktree, including for gate-slot limits and on adopting machines.
  Refusing there would stop every gate and Drift call, even ones that never run the command.
  At run time, Drift's check before it spawns and the gate's existing "could not start the shell" refusal cover a `dir` that is missing in a task worktree.
- The gate's own run path keeps `WithDefaultDir` with its `resolveRunDir`.
  Together they compute the same result as `ResolveCommandDir`, so this change leaves that code alone.

## Tests

- `gateslot`: `TestResolveCommandDir` covers each resolution case.
  `TestCheckDirs` covers refusing a missing tier or build `dir` by key and resolved path, and accepting existing and unset dirs.
- `driftengine`:
  - New rows in `TestRunCapture` run under an anchor at `dotnet`. They show that an unset `dir` runs in the anchor and that `dir: dotnet` runs in `<root>/dotnet`, the case that used to double. They also show that a missing `dir` is refused with the key, the path and the way forward, before any slot or log is taken.
  - A new `TestRunCompare` row covers the compare's own way forward.
  - `TestCheckDirs` covers the config-level check.
  - Existing config tests now expect an empty `dir` where they used to expect `"."`.
- `configcli`: three new `TestDispatchHubWideBoard` rows.
  A drift `--set` of a missing `dir` is refused and the file is restored.
  A drift `--set` of an existing `dir` is written.
  A gate `build.dir` `--set` of a missing `dir` is refused.
