---
format: 5
approved: false
language: go
---

# Plan: KDE Plasma 6 window placement

On KDE Plasma 6 under Wayland, every terminal window lym opens for a worktree, from `lym reed window` and from `lym machine start`, is handed to a detached placement child.
The child places the window on a desktop the remembered places or the `same`/`next` mode choose, builds that desktop's KWin tiling grid and snaps its windows into it.
A new package, `internal/kwinplace`, is the only code that reaches KWin and kscreen.
On any other desktop, session or platform, a window opens as it does today.

## Card Index

1 — beam-window-placement-setting — add the per-task beam.window_placement setting and its lookup
2 — desktop-placement-keys — add desktop.window_placement and the two smallest-tile machine keys
3 — window-command-cancel — run the window command with no wait under a cancellable context and process group
4 — konsole-separate-launch — start Konsole with --separate and return its process id
5 — kwinplace-scripts-readers — create kwinplace with its KWin scripts, snapshot types, readers and recordings
6 — kwinplace-prelaunch — add the pre-launch checks and the one window opener
7 — kwinplace-script-runner — run scripts through qdbus6 and the journal under the per-user script lock
8 — kwinplace-places — keep the per-hub remembered places file and its update rules
9 — kwinplace-planner — plan the target desktop, grid and window order from a snapshot
10 — kwinplace-place-flow — run the placement child's waits, locks, apply, record and window command
11 — place-window-child — spawn the detached child through the hidden lym reed place-window verb
12 — reed-window-placement — hand lym reed window's window to the placement child
13 — machine-start-placement — hand lym machine start's windows to the placement child

## Shared Decisions

### Decision: probes-before-code

- **Decision:** Before card 1, the implementing run runs the KWin probes the decision record's Open risks list, as that section prescribes: the read-only ones itself where its session reaches the user session bus, the rest through its parent and the operator.
  A fact a probe contradicts stops the run before any card, and the run sends the probe's command and output to its parent.
- **Rationale:** the decision record makes this the run's first step; no card carries it, so no card's work depends on a fact found while it runs.
- **Applies to:** all cards

### Decision: tier0-for-pure-rules

- **Decision:** The planner, the places rules and the flow are tested at `tier0`, over kwinplace's own types, hand-built, with the runner's KWin calls behind function fields the runner fills.
- **Rationale:** the record's acceptance criterion asks for the rules under tier-1 tests; `PATTERN-test-speed` puts a behavior at the lowest tier its real modules run in, and these run in memory and in temp dirs, so the constraint wins.
  Only the `qdbus6`, `journalctl` and `kscreen-doctor` answers come from recordings (`PATTERN-test-stand-ins`).
- **Applies to:** cards 5, 7, 8, 9, 10

### Decision: contract-test-tier

- **Decision:** The contract test against a real KWin is `tier3`.
- **Rationale:** the record asks for the tier that runs a real desktop, whose run is the operator's call; `tier3` is the hub's only tier no gate runs, run by hand.
  `PATTERN-test-speed`'s table names `tier3` for a real LLM; no tier names a desktop, and a gated tier would fail on every machine without one.
  Card 7 adds a real desktop session to that table's `tier3` row, so the background stays true.
- **Applies to:** card 7

### Decision: run-registration

- **Decision:** The placement child of every worktree but the prime registers in `internal/runproc` under the worktree's name with `RolePlacement`.
- **Rationale:** D10 registers it "when the opening verb runs inside a run"; a run's and a beam's `End` work by the worktree's owner name, so registering by name lets every run or teardown end reach its child with SIGTERM, which D12 relies on. The prime's window belongs to no run.
- **Applies to:** card 11

### Decision: hub-geometry-in-the-verb

- **Decision:** The hidden verb resolves the mode, the parent and the hub's worktrees, and hands them to `kwinplace` in its `Request`.
- **Rationale:** `kwinplace` then needs no hub geometry, and its flow stays testable over its own types.
- **Applies to:** cards 10, 11

### Decision: carry-over-findings

- **Decision:** The Discussion-Review carry-over entries in the record's Open risks are not planned as scope; the record's D2, D3, D6 and D9 text, which the round-3 fixer revised, is what the cards follow.
- **Rationale:** the plan writer's instructions say never to plan a carry-over entry.
- **Applies to:** all cards
