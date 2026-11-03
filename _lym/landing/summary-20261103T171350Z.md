# Merge origin/main into the codex-engine beam

Brings the `codex-engine` beam up to date with `origin/main` at `d026a9a8`, so the beam's landing merges into main cleanly.
No new behaviour: both main's changes and the beam's Codex work stand unchanged.

## Why the merge base matters

The beam's earlier merge commit `9447bbda` holds main up to `f5724733` in its tree, but it has one parent, so main's history was not an ancestor of the beam.
A plain `git merge origin/main` therefore took the fork point `7f0f65b3` as its base and reported conflicts in `docs/overview.md` and the `hibernatecli` and `hibernateengine` packages, where both sides held main's own changes.
The beam's own commits never touched the hibernate files, and the beam's copy of each matched `f5724733` exactly.

This merge uses `f5724733` as its base instead.
With that base the three-way merge is clean: the result is the beam's tree plus main's diff from `f5724733` to `d026a9a8`, with no hand-resolved hunk.
The merge commit's second parent is `d026a9a8`, so main's history is now an ancestor of the beam and later merges find the right base on their own.

## What main brings in

- `hibernatecli` and `hibernateengine`: the prime's dobby carries a hibernate request through a new carrier, replacing the detached `lym hibernate watch` daemon; `beamcli` and `fabriccli` wire it in.
- `driftcli`, `driftengine` and `loomcli`: one base for a command's dir, and a missing Drift dir fails by name.
- Smaller follow-ons in `battencli`, `cleanupcli`, `configcli`, `configreg`, `gateslot` and `lymdirs`, plus the matching `docs/overview.md` and `pattern/PATTERN-daemon-wakeups.md` updates.

## What to check

- `git diff 9447bbda 7d7fdf87` and `git diff f5724733 d026a9a8` add and remove the same lines with the same `--numstat`, which confirms that the merge adds main's changes and nothing else.
  Only hunk offsets in `docs/overview.md` differ, because of the beam's own lines there.
- `go build ./cmd/lym` and `go vet` over every package main changed, along with `shuttleengine`, `codexengine`, `claudeengine`, `stencilstore`, `heddleengine` and `shellguard`, pass on the merged tree.
  The gate has not run yet; the verify gate runs it.
