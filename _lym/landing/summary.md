# Merge main into the codex-engine beam and resolve every conflict

Brings `origin/main` into the `codex-engine` beam a second time, so the beam's landing merges cleanly.
The merge adds no behaviour of its own: it reconciles main's changes with the beam's, and fixes what the merged tree broke in the build and the gate.

## What changes

Two sets of changes meet in the merged tree.

Main's change: a hub command's `dir` resolves against the worktree root, not the anchored directory.
- `gateslot.ResolveCommandDir` and `gateslot.StatCommandDir` are the one rule: an empty dir is the anchor, an absolute dir is itself, any other dir is joined onto the worktree root.
- The drift capture and compare steps run through it.
  `CaptureRequest`, `CompareRequest` and the step run carry `WorktreeDir` next to `AnchorDir`.
  A step whose dir is no existing directory is refused before it waits for a slot, with the config key, the resolved path and the way forward.
- `drift.yaml` no longer defaults a step's dir to `.`, so an unset dir runs in the anchor as the gate's does.
  The Drift-Baseline prompt names the resolved directory, relative to the worktree root when it lies under it.
- A new `configreg.Module.CheckDirs` hook, set for `drift` and `gate`, lets a hub-wide `lym config` write refuse a dir that is no existing directory in the prime.
  `configcli` runs it through `checkHubModule` in `lym config --set` and in the edit path; `newLoadFailure` now takes the check's error instead of loading the module itself.
- `battencli` hands the task's worktree path, not its anchor, to the dobby notice and the pair dir.

The beam's change: the hibernate request is carried by the prime's dobby.
- `lym hibernate request` writes the request and returns; it no longer spawns a daemon, and the hidden `lym hibernate watch` verb is gone.
- A new `hibernateengine.Carrier`, run from the prime's dobby tick, takes up a pending request no live process carries and runs its sequence in a goroutine.
  A request taken up in the pausing or confirming phase has its pause entry lifted and starts again from the trigger.
  One taken up in the hibernating or resuming phase fails at once, since its wake can no longer be seen.
- `lym hibernate request` reports `dobby_live` and, when the prime's dobby is down, the way forward `lym beam start root`.
- `beamcli` wires the carrier into the prime's dobby only, watches the request file, and holds a re-exec of the binary while a sequence is in flight past its trigger wait (`holdReexec`).
- `lym hibernate cancel` no longer kills a process; the sequence sees the cancel at its next step and lifts what it added since (`withdrawn`).
- `lym anchor` refuses a prime move only while a hibernate request is pending in a running prime dobby.

Docs and comments follow both sets: `docs/overview.md`, the package docs and `pattern/PATTERN-daemon-wakeups.md`.

## Design decisions

- One resolution rule for a command dir, in `gateslot`, shared by the drift steps, the baseline prompt and the hub-wide checks, so one value means one directory in every config file.
- `fabriccli` cannot import the hibernate packages, so its list of ended request phases is a local copy; a test pins it to `hibernateengine.TerminalPhases`.
- `CheckDirs` runs only against the prime, since a hub-wide write runs only from there.

## Check by hand

- The merged tree's build and gate were not run in this description step; run `lym gate test` over the touched packages before landing.
- Start the prime's dobby, run `lym hibernate request`, and confirm the dobby takes the request up, including after a dobby restart.
- Set a missing dir with `lym config drift --set capture.dir=<dir>` and `lym config gate --set build.dir=<dir>` from the prime, and confirm the write is refused.
