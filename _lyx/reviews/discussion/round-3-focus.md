---
round: 3
focus:
  - "Attach-time window selection: in `internal/reedengine/attach.go`, `AttachArgv` returns `bareAttachArgv` when the client size is unknown, when the window-size or status readback fails, when the strands' window holds fewer than two live panes, or when no placed strand is live. Check that `_lyx/discussion/decision-record.md` D5's Attach bullet gives a selection of the relevant window that is applied on every one of those returns, not only on the layout-chain path. Settle whether the common session, where the overview, driver and orchestrator sit outside the shell window and Selvage is alone in it, still selects the newest live agent's window on attach."
  - "Migration placement: check that D10's rename and move step runs in the attach pre-flight in a position that no early return of `AttachArgv` (unknown client size before the lock, readback failure, fewer than two panes, no placed strand) can skip, and that the acceptance criteria cover the attach path where Selvage is the only pane in the shell window."
  - "Failure logging under `PATTERN-log-levels` (the `internal/logger` Level policy: a repeating failure logs Warn once per episode and Debug while it persists): check that D10's logging of a failed rename or move, retried at each window open and attach, follows it and agrees with D2's Debug for refused `-b` and `-a` insertions on psmux."
  - "Order premise on psmux: check that every decision that relies on index order meaning start order, D5's relevant window and D6's conditional step, states the best-effort caveat that D2's plain `new-window` fallback on psmux imposes, where a new window takes the first free index."
---

The focus targets the attach path, where the design stakes lie, and the two logging and psmux-order consistency points.
