# Agent probing 'lyx <verb> --json' as help ran destructive leaf verbs (fabric unwire) in a task pair

`json-leaf-runs-verb` · Note · bug

Imported from [issue #512](https://github.com/Knatte18/loomyard/issues/512).

## Seen

On 2026-10-10 at 12:32Z the Plan-Review reviewer of task `cli-surface` walked the command tree in its pair worktree with a shell loop: `walk() { j=$(lyx $path --json); for c in $(jq -r '.commands[]?.name'); do walk "$path $c"; done; }`.
It had checked that `lyx board --json` prints a group's command tree, and assumed the same of every node.
For a leaf, `lyx <verb> --json` runs the verb, so every leaf that needs no argument ran for real in the pair, about 33 verbs in 8 seconds.

Mutations from the traces:

- `lyx fabric unwire` removed the pair's `_lyx` and `.lyx` links; the batten run's Run-Shed then failed on a missing child status file, and later processes recreated `.lyx` as a plain directory in the warp.
- `lyx board retire-legacy` deleted the board's legacy `notes.json`, `tasks.json` and the `legacy_done` list (board commit bac875f2).
- `lyx fabric push` and `lyx fabric sync` pushed the pair's weft branch; `lyx fabric checkout` re-switched both sides (no-op); `lyx board prune` and `rerender` spawned board syncs.

## Workaround

`lyx fabric reconcile` in the pair recreated `_lyx`, but refuses to re-point `.lyx` because a trace log exists in both the warp's stray `.lyx/logs` and the weft's; the warp-side copy must be moved away by hand before reconcile succeeds.

## Expected

- A destructive verb that takes no argument (`fabric unwire`, `board retire-legacy`, `board prune`) does not act on a bare invocation from an agent probing the CLI, for example by requiring an explicit confirmation flag; or `--json` on any command never doubles as "describe".
- Command-tree discovery has one documented read-only route (such as `lyx help --json <path>`) that agents can rely on for leaves and groups alike.
