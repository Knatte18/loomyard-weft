---
verdict: APPROVED
---

## Scope against the board entry

The entry `tender` lists candidate ideas and leaves to Discussion which belong in the module; it carries no separate Done-when, so the goal, the two folded-in sections and their Scope lists are the yardstick.

- Module and shortcuts: D1 and D2 move `pause` and `resume` into `tender` and keep `lyx pause` and `lyx resume` as one-line shortcuts, as the Idea asks.
- Revive: D5 covers the outage revive the entry describes, `idle-stalled` with cause `api-error`, one line per session.
- Status: D3 gives one view of every live session in the hub.
- Drain and the concurrent-run cap are left out with reasons; the entry offers them as candidates to discuss, not as requirements, and the cap stays an open point of `orch-split`, so nothing is duplicated there.
- Issue #606: D6 covers both Scope points, the held time no longer counting against the deadline and a deadline error staying transient and re-stepped rather than halting the run `failed` on first occurrence.
- Stale binaries: D8 and D9 make `lyx orch watch` and `lyx reed attach`, the two processes the entry saw, stop running an old binary; D7 lists the rest hub-wide.
  The Scope Out drops a restart verb, which narrows the title's "restart"; the entry itself offers self-re-exec as one of its two mechanisms and asks that processes which must not restart mid-work only be reported, so the narrowing is one of the entry's own options.
- The reed watchdog and status panes, which the entry says moved to `reed-window-order`, are left untouched.

## Related board entries

- `orch-split`: owns the concurrent-run cap; the design leaves it there.
- `creel`: plans to own pane delivery; revive reaches panes through the same seams `pause` and `resume` use, so creel's later wiring replaces all three together, with no contradiction.
- `spawn-audit-log`: plans a recorded launch path and a process sampler; D7's narrow stale-binary reading and D9's exec are named as later covered by it, with no duplicated verb or log.
- `weaver`: only a name the entry rules out; no overlap.

## Overlaps with runs in flight and parent commits

`lyx loom peers` lists no peer run and no parent commit (`own_row` 3), so there is no overlap and no landing order to give.

Shared bookkeeping the design touches, which does not escalate: `cmd/lyx/testdata/test-budget.yaml` (renamed `pausecli` and `pauseengine` rows), the module table in `docs/overview.md`, and the `pausecli` deviation line in `pattern/PATTERN-cli-cobra.md`.
