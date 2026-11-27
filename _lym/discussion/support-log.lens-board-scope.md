```yaml
lens: board-scope
outcome: reported
```

Board-scope lens: checking the decision record's fit with the board entry, the touching entries, the peers and the parent commits.

**Findings**

1. MEDIUM (`decision`), D7 / Scope / Open risks: the record drops a goal point the operator already decided.
   - The entry says, under "Decided (operator, 2026-11-27)": "The hub sets the default per machine, and a beam or project on the board may override it."
   - D7 gives the override to task and beam worktrees only, and the Scope's "Out" list excludes a project that is not a beam.
   - The board does refuse `run_settings` on a project entry, so the writer cannot settle this alone, and it is correctly raised in Open risks.
   - Question for the parent: should this run let the board accept a placement setting on a project entry, or does the operator accept beam-only overrides as the reading of "beam or project"?

2. LOW (`decision`), Scope / Open risks: VS Code windows are left out, which narrows the entry's title.
   - The title says "Windows lym opens…", and the wish says "for each window it opens".
   - The record excludes VS Code from `lym ide` and from `lym machine start`'s workspace launch. Its reason is that VS Code runs every window in one process, so a process-id match fails.
   - The parent question is already in Open risks. The wish's context is the windows the operator tiles across desktops, the run and orch terminals, so terminal-only is a plausible reading.
   - Question for the parent: are terminal windows alone the intended scope?

No other contradiction with the board.

**Goal points of the entry**
- Remember each window's desktop and tile, and return there after a restart or `lym machine start`: met (D8, D9).
- A new window is placed by rules the operator sets in config: met (D7, D8).
- The "start every hub" launcher becomes useful: met (D10 lock, D12 through `lym machine start`).
- Plasma 6 only, with other desktops and Plasma 5 working as today: met (D1).
- Tiling uses KDE Custom Tiling grids, built the way the operator's scripts build them: met (D3, D6).
- lym chooses the grid, sets it and snaps the window, going further than an auto-snap: met (D6).
- `same` and `next` relative to the parent orch's desktop, with `next` overflowing: met (D8).
- Hub default per machine: met (D7). Beam override: met (D7). Project override: not met (finding 1).
- One screen only: met (D4).
- The grid follows the screen's resolution and scale, the laptop takes fewer windows, at most 2x5 and ten windows per desktop: met (D5).
- Proposed: the priority-1 screen: met (D4). The smallest tile size in logical pixels: met (D5).
- To settle: grid growth: met (D6). Screen-setup change: met (D11). Where places live: met (D9, machine state). Where the auto-snap script lives: researched and not found, recorded in Constraints.
- The title's "every window lym opens": partly met (finding 2).

**Entries, peers and commits checked**
- `area-host-exits` (T16, draft, under the closed-area project): no conflict.
  - It moves Konsole launching behind a supervisor window verb inside an area, and the record's Open risks already names the process-id handover.
  - The two touch `termwindow` in a text-only overlap. T16 depends on the area chain and lands later.
- The closed-area project (P7): no conflict.
- Peer `fabric-core-foundation`: no conflict. It changes files in `internal/hubgeom`, `internal/machinecli` and `internal/reedcli` as part of its layout retarget, a text-only overlap of the same packages. Plan should rebase on it if it lands first.
- Peers `scope-and-earlier-checks`, `finalize-fixer-halt-blame` and `remove-testmap`: no conflict, and no overlap with the window packages.
- Parent commits: `lym loom peers` listed none.

**Decision notes**
- No `Decision for kde-window-placement:` note is on the board, and the record has no `Added after Discussion` entries. I reached every note there was.

Out of my lens: D13 edits only the background of `PATTERN-config-layers`, but that entry's Rule line names the per-task beam keys one by one. A `## PATTERN changes` bullet may be needed.

**Tool calls**
- I made one call: I grepped the saved `lym loom peers` output for changed paths per peer. My context held the peer slugs, but not which peer changes `internal/hubgeom`, `internal/machinecli` and `internal/reedcli`.
