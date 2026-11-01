# Card 1 — pattern-entries

**Edit:**
- `//PATTERN.md`
- `//pattern/PATTERN-durable-vs-ephemeral-state.md`
- `//pattern/PATTERN-config-layers.md`
- `//pattern/PATTERN-cwd-resolution.md`
- `//pattern/PATTERN-fabric-git.md`

**Extends:**
- `//pattern/PATTERN-leaf-packages.md`

**Create:**
- `//pattern/PATTERN-heddle-instructions.md`

**Intent:** Bring `PATTERN.md` and its background files in line with the task's declared PATTERN changes, ahead of the cards that rely on them.

In `PATTERN.md`, replace the `PATTERN-durable-vs-ephemeral-state` line under `## Paths and geometry` with this whole line:

```
- `PATTERN-durable-vs-ephemeral-state` — Adding file: tracked under `_lym`, never-tracked under `.lym`, same subpath, bar heddle's rendered instruction files and their `.heddle` temporary directory at a warp root; only `internal/lymdirs` names either, bar a `tools/` script's variable block. — [background](pattern/PATTERN-durable-vs-ephemeral-state.md)
```

Keep the `PATTERN-leaf-packages` line as it stands; the task declares it changed only to record its background edit, and its overview line is this one:

```
- `PATTERN-leaf-packages` — Importing into a leaf package: only closed import set. — [background](pattern/PATTERN-leaf-packages.md)
```

Add this line as the last entry of `## Agents and prompts`:

```
- `PATTERN-heddle-instructions` — Rendering heddle entries into a worktree file or a session's system prompt: only from `heddle.json` in the Bolt checkout, through `internal/heddleengine` (new); that file written only by `lym heddle` verbs and heddle's Bolt replayer; fabric names no heddle file. — [background](pattern/PATTERN-heddle-instructions.md)
```

In `pattern/PATTERN-durable-vs-ephemeral-state.md`, add a bullet stating the exception: heddle's rendered instruction files (`CLAUDE.local.md`, `AGENTS.override.md`) and the `.heddle` temporary directory sit at a warp worktree's root, outside `.lym`, untracked through the warp repository's `info/exclude` lines fabric seeds from its exclude-list key, because the agents' instruction loaders read only the worktree's root and a rename onto the root must not cross a filesystem.

In `pattern/PATTERN-leaf-packages.md`, extend the `internal/runproc` section's import set with `internal/lymcwd`, admitted for the one Bolt directory name `runproc.HubOwner` takes from it.

Create `pattern/PATTERN-heddle-instructions.md`: heddle's store is `heddle.json` at the Bolt checkout's root; the `lym heddle` verbs and heddle's Bolt replayer are its only writers; `internal/heddleengine` is the only package that reads it and renders it, into file targets at a worktree's root through fabric's worktree seam or `lym heddle render`, and into a session's appended system prompt through shuttle's prompt seam; fabric hands heddle paths and events and names no heddle file.

Background lines that place a Bolt-checkout thing through `BoardDir` or `_board` name `BoltDir` or the Bolt checkout instead (D9), wording otherwise unchanged:
`PATTERN-config-layers.md`'s machine config file becomes `<BoltDir>/.lym/config/<module>.machine.yaml`;
`PATTERN-cwd-resolution.md`'s hub-only containers line names the Bolt checkout and `_launchers`;
`PATTERN-fabric-git.md`'s one weft-side `.gitignore` writer is `boardengine`'s lock and manifest patterns in the Bolt checkout, the weft's `main` worktree;
and `PATTERN-durable-vs-ephemeral-state.md`'s hub sibling is `BoltDir(hub)`.

The overview grows by the new line and the longer durable line, well under the headroom line (the overview is far below it today).

**ImpactSummary:** Prose only: three PATTERN lines and the background files listed above; no code or test reads the changed wording.
