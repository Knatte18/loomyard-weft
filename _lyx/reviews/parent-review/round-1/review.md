---
verdict: APPROVED
---

## Scope against the board entry

The decision record meets every point of `umbrella-daemon`'s entry, the operator decisions of 2026-10-18, 2026-10-19, 2026-10-24 and 2026-10-25, and the parent's arbiter decision of 2026-10-25:

- One dobby per beam, the prime included, in the beam module, with creel sender `DOBBY`, started by `lyx beam start`, which `lyx orch start` calls first, and never at machine start: D1, D2, Scope Out.
- Dobby follows status files through fswatch, with the probe timer as the fallback where file watching fails: D4, D13.
- Dobby starts ready tasks, drives their batten runs through Land and teardown, and revives dead drivers, while batten keeps its rows and knows nothing of its owner: D3, D4, D9; the entry's optional step 2 is out of scope as the operator decided.
- Mechanical stops go through the rule table, tackler and arbiter, then the nearest orch: D7, D8.
- The orch never starts, stops or cleans up a run, and the `newly ready` part of the landing notice goes: D6.
- Draft is a status value, not a field, and readiness is computed from the board: D5.
- Every batten notice is revised: D9.
- Tender stays the brake; dobby honours its pause entries, checks covered runs after a lift, and revives API-error stalls on its own while `lyx tender revive` stays manual: D9, D12, Scope Out.
- The board tidy every N landings, N defaulting to 5, in a window of the beam's reed session, its report handed to the orch: D10.
- The key-level per-worktree config check, with the three behaviours the operator listed: D11.
- The arbiter seam moved into shed for every shed-driven recipe, its keys in `shed.yaml` with module overrides, and a description of batten's rows: D8.
- "umbrella" is renamed "beam" in every text the task touches: Scope In.

Two points were weighed and are not scope deviations:

- D11 runs the config check in Finalize rather than in dobby, where the operator's 2026-10-13 note placed it in "the Go run driver".
  The record names the departure and its reason, the three behaviours the operator asked for are all met, and where a Go step runs the check is a design detail for the Discussion review.
- D4 retires `--window` and the hidden `--open-terminal`, which the entry did not ask for.
  Both follow from dobby starting every run's passes detached, and D6's removal of the orch's `--window` start step, which the entry asks for, leaves `--window` with no caller.
  The record names the retirement as an addition.

Entries the record leaves out on purpose, each its own work: `umbrella-anchor`, `loom-beam-darns`, `landing-escalate-arbiter`, and the hub-wide orch rule "Board tidy after a landing", a Bolt write for the parent.

## Superseded

`batten-window-default` asks that `lyx batten run` start in its own window by default.
D4 retires the window flag, makes the open_window setting alone decide the Open-Window row, and removes the note with `lyx board remove`.

## Overlaps

Parent commits: `lyx loom peers` lists none since the fork.

Peer `discussion-scope-in-lenses`, at row 3, the same as this run's `own_row` 3, so the landing order is a tie.
What overlaps, with no design conflict:

- The orch role stencil: the peer removes its Parent-review section, and this run removes the start steps and window wording; different sections.
- `internal/arbiterstop`: the peer adds a refusal of a `decision` escalation, handed back to the parent; this run adds a batten eligibility rule and leaves loom's and darn's rules unchanged.
  The eligibility stays in `arbiterstop` under D8, so the peer's refusal survives the arbiter seam's move into shed, as the peer's Open risks requires.
- The `Merge-In` to `Parent-Align` rename in texts `arbiterstop` and `sheddriver` hold: this record's Constraints already name it, and whichever run lands second takes the other's wording.
- `loom.yaml` reconcile: the peer drops the alignment and parent-review keys, and this run moves the arbiter keys to `shed.yaml` while keeping loom overrides; different keys.
- Batten's Run-Shed producer: the peer removes its `ReviewWait` seam, and this run turns its wait into passes; different parts of the same producer.
- PATTERN: this run adds `PATTERN-dobby-starts-runs` and `PATTERN-shed-stop-chain`, and the peer changes no PATTERN entry.

Board entries checked with no overlap beyond what is named above: `idle-notice-dead-waiter-cause`, whose waiter is a shuttle run's waiting process, not batten's; `spawn-audit-log`, named in Open risks; `creel-agents` and `run-verbs-shared`, which change no behaviour this record decides.
