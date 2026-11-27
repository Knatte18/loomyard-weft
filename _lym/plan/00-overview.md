---
format: 5
approved: true
language: go
---

# Plan: KDE Plasma 6 window placement

On KDE Plasma 6 under Wayland, every terminal window lym opens for a worktree, from `lym reed window` and from `lym machine start`, is handed to a detached placement child.
The child places the window on a desktop the remembered places or the `same`/`next` mode choose, builds that desktop's KWin tiling grid and snaps its windows into it.
A new package, `internal/kwinplace`, is the only code that reaches KWin and kscreen.
On any other desktop, session or platform, a window opens as it does today.

## Card Index

1 — kwin-probes — run the KWin probes and take the KWin, qdbus6 and kscreen recordings the kwinplace tests read
2 — beam-window-placement-setting — add the per-task beam.window_placement setting and its lookup
3 — desktop-placement-keys — add desktop.window_placement and the two smallest-tile machine keys
4 — window-command-cancel — run the window command with no wait under a cancellable context and process group
5 — konsole-separate-launch — start Konsole with --separate and return its process id
6 — kwinplace-scripts-readers — create kwinplace with its script fillers, snapshot types and readers
7 — kwinplace-prelaunch — add the pre-launch checks and the one window opener
8 — kwinplace-script-runner — run scripts through qdbus6 and the journal under the per-user script lock
9 — kwinplace-places — keep the per-hub remembered places file and its update rules
10 — kwinplace-planner — plan the target desktop, grid and window order from a snapshot
11 — kwinplace-place-flow — run the placement child's waits, locks, apply, record and window command
12 — place-window-child — spawn the detached child through the hidden lym reed place-window verb
13 — reed-window-placement — hand lym reed window's window to the placement child
14 — machine-start-placement — hand lym machine start's windows to the placement child

## Shared Decisions

### Decision: probes-before-code

- **Decision:** Card 1 runs the KWin probes the decision record's Open risks list and takes every recording the kwinplace tests read,
  ahead of every card whose code rests on a KWin fact.
  The read-only ones run in the implementer's session where it reaches the user session bus; the moving ones are asked of the operator through the parent in card 1, before any later card starts.
  A fact a probe contradicts halts the run for the parent before card 1's commit.
  Cards 6 and 8 read the recordings card 1 committed and carry no operator step.
- **Rationale:** the record's added decision "A first card runs the KWin probes" puts every probe and every operator step in one card ahead of the code that rests on the facts.
- **Applies to:** cards 1, 6, 8

### Decision: tier0-for-pure-rules

- **Decision:** The planner, the places rules and the flow are tested at `tier0`, over kwinplace's own types, hand-built, with the runner's KWin calls behind function fields the runner fills.
- **Rationale:** the record's acceptance criterion asks for the rules under tier-1 tests; `PATTERN-test-speed` puts a behavior at the lowest tier its real modules run in, and these run in memory and in temp dirs, so the constraint wins.
  Only the `qdbus6`, `journalctl` and `kscreen-doctor` answers come from recordings (`PATTERN-test-stand-ins`).
- **Applies to:** cards 6, 8, 9, 10, 11

### Decision: contract-test-tier

- **Decision:** The contract tests against a real KWin are `tier3`.
- **Rationale:** the record asks for the tier that runs a real desktop, whose run is the operator's call; `tier3` is the hub's only tier no gate runs, run by hand.
  `PATTERN-test-speed`'s table names `tier3` for a real LLM; no tier names a desktop, and a gated tier would fail on every machine without one.
  Card 8 adds a real desktop session to that table's `tier3` row, so the background stays true.
- **Applies to:** card 8

### Decision: run-registration

- **Decision:** The placement child of a worktree other than the prime registers in `internal/runproc` under the worktree's name with `RolePlacement`
  only when the opening verb runs inside a run, which a non-empty `pauserecord.RunProcessEnv` in its environment marks.
- **Rationale:** D10 registers it "when the opening verb runs inside a run", and a run command sets that variable for itself and every process it starts.
  A run's and a beam's `End` work by the worktree's owner name, so registering by name lets every run or teardown end reach its child with SIGTERM, which D12 relies on.
  A child opened outside a run, and the prime's, belong to no run and end within their own bounds.
  Card 12 lists how each reader of a worktree's runproc records counts a registered child.
- **Applies to:** card 12

### Decision: hub-geometry-in-the-verb

- **Decision:** The hidden verb resolves the mode, the parent and the hub's worktrees, and hands them to `kwinplace` in its `Request`.
- **Rationale:** `kwinplace` then needs no hub geometry, and its flow stays testable over its own types.
- **Applies to:** cards 11, 12

### Decision: carry-over-findings

- **Decision:** The Discussion-Review carry-over entries in the record's Open risks are not planned as scope; the record's D2, D3, D6 and D9 text, which the round-3 fixer revised, is what the cards follow.
- **Rationale:** the plan writer's instructions say never to plan a carry-over entry.
- **Applies to:** all cards
