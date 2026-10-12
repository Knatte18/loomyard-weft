---
verdict: APPROVED
---

## Scope against the board entry

The decision record covers every point of the `reed-multi-window` entry and adds nothing outside it.

- Own-window anchoring as a `display` anchor: D1.
- The producer as the unit, a few agents sharing one window and many getting one each: D1, D2 and D11.
- Shared windows split by height or by width: D3.
- Strands in depth, the driver and its `loom-status` strand staying in window 0, each spawned agent in a window named by its role: D2 and D10.
- The window closing when the agent ends, with the view falling back to the driver: D5.
- No focus stealing, and the open question of how a new window announces itself: D4.
- The folded `extra-agent-strand`, a named LLM session through shuttle with no turn, its session id recorded for the watchdog's name repair, and the verb kept out of reed: D12.
- The two dropped ideas, cross-worktree columns and per-window attach through session groups, sit under Scope Out.

Two readings depart from the entry's literal wording and stay within its scope:

- The entry says the split direction is "set the same way" as the share choice; D3 puts it in `reed.yaml`, and D2 has reed apply a cap from `reed.yaml` to a group size the strand's Display declares.
  The capability the entry asks for is there; where the setting lives is a design detail for Discussion-Review.
- The entry says an own window "replaces zooming a pane"; the record keeps zoom for window 0 (Q7) and gives spawned agents own windows, which is that replacement for them.
  The entry asks for no removal of zoom.

## Other board entries

- `run-window-bugs` depends on this task and keeps the readiness probe fix; the record leaves it there (Scope Out) and hands it the open risk of a detached window's default size.
  That risk is not yet written in the `run-window-bugs` entry; the orch may want to add it there.
- `seats-replaces-singlellm`: D11's one-seat table named by its seat's role matches that entry's naming rule.
- `creel`, `orch-split`, `orch-any-worktree` and `discussion-multi-llm-bench` neither duplicate nor contradict any decision here.

## Overlap with runs in flight and the parent

- `cleanup-1012` changes `internal/fabricengine`, `internal/lyxcwd` and `tools/tokencount`; nothing this design touches.
- `pause-esc-rewind` is at Discussion-Write with no changed paths and no decision record.
  It will likely change pause and resume code (`internal/pauseengine`, possibly `internal/shuttleengine/claudeengine`), and this design reads the pause in D8 and D12 and puts D12's provider specifics in `claudeengine`.
  That is no overlap today; the check at PR-Gate should look again once that run has a record.
- The parent's landed commit `22b4aff` changes `internal/gatecli`, `internal/planparser`, `internal/shedverbs`, `internal/shedengine`, `PATTERN.md`, `pattern/PATTERN-cli-cobra.md`, `pattern/PATTERN-failure-stop-visible.md`, `pattern/PATTERN-glyph-conversion-chokepoint.md` and `docs/overview.md`.
  This design proposes no change to any PATTERN entry; its Constraints only cite them.
  A module-table change in `docs/overview.md` would be shared bookkeeping, not an overlap.
