```yaml
lens: board-scope
outcome: reported
```

Task: board-scope review of the gate-builds-testmap decision record against the board.

I found no BLOCKING or MEDIUM findings.

**Findings**

- **LOW — D1 / Goal:** the entry says the map is "a mode of the gate's own tier 0 and tier 1 run". The record runs it as its own pass: it skips the tiers' shell commands and refuses `--tier` and `--upto`. It still keeps the flag on `lym gate test`, the tier env, the tier parallelism and the gate slots, and the support log's rejected alternatives explain why. This reads as a fair interpretation, not a contradiction. Evidence: the entry body's Idea bullet 1, against D1 and D4.
- **LOW — Open risks (consistency):** two other entries still name `lym gate testmap` in their bodies: `redundancy-sweep-existing` (Idea point 4 and the Seen paragraph) and the project `test-suite-trim` (Order item 1). The record mentions only the first, as an open risk. It is no parent question, because that run reads the new flag on its own. The project body's wording is historical bookkeeping and conflicts with nothing. Evidence: the `lym board get` outputs for both entries, already in context. A possible fix is to drop the Open risks line or keep it as a note; nothing needs the parent.

**Goal points of the entry**

1. A flag on `lym gate test` sets map mode, and the `lym gate testmap` verb goes: met by D1 and D9.
2. Map mode builds with `-cover`, runs one process per test with its own profile, and `testcover` merges at package end: met by D3.
3. The test processes run in parallel within the gate's own parallelism: met by D4.
4. Risk of shared state outside the process: met by D5 and the Open risks line.
5. Risk that the memory guard and per-run cap must cover the processes: met by D4. The per-test caps are clamped to the tier's `memory_mb` and the guard samples lym's own tree. The builds outside the cap are a stated bound.
6. Risk that a `-cover` build doesn't share the normal build cache: met in Constraints, with the smart-build study at Go 1.26.8.
7. Map uses are decided apart: met, since Scope Out excludes them.

**Entries, peers and parent commits checked**

- `redundancy-sweep-existing`: no conflict. It waits on this task and builds maps fresh per worktree, which D8 keeps.
- `test-suite-trim` (project): no conflict. It already holds that maps are built only on command and the gate selects nothing from them.
- `readonly-test-leftovers` (note): no conflict. Guard kills in map mode can leave read-only leftovers, but that note owns the sweep fix and the record doesn't touch it.
- Peer `machine-registry-holds-catchup`: no conflict. It changes `internal/testkit/tmuxkit/tmuxkit.go`, which D5's isolation premise relies on. That is a text-only overlap; nothing in the peer's title or paths touches per-process run directories.
- Peer `codex-standing-instructions`: no conflict, and it changes no paths.
- `lym loom peers` lists no parent commits.

**Decision notes**

The board list shows no note titled `Decision for gate-builds-testmap:`, and the record has no `Added after Discussion` entries.

**Tool calls**

I made none: the board list, this task's entry, the touching entries and `lym loom peers` were all already in context.
