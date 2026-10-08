---
round: 4
focus:
  - "D7 in `_lyx/discussion/decision-record.md` (the top-pin adjustment): settle which pane D7 means by 'the pane at window row 0' and where reed learns it, given `render.FixedHeightPins` takes no `paneOrder` and `Rules` places the physically first pane at row 0 after a reorder; check the acceptance criterion covers strand panes in a non-intended physical order."
  - "D7 against D6: D6 makes the `pane-border-status top` set non-fatal, while D7 subtracts the title row from the top pin; settle what D7 does when the border status was refused or its readback fails, and whether D7 keys on the effective status as it does for `#{status}`."
  - "D7 at the lower bound: the height planner floors every cell at one row; settle what D7 states for a planned one-row top cell, where the adjusted pin would be `-y 0`."
  - "D12 (teller input and the 'recorded and not live' test): settle which side decides whether a predecessor's reed state records a session that is not live, and whether the per-entry teller input (name, spawn-order key, up function) supplies what that side needs, given `PATTERN-told-geometry` and that reedengine reads no loom or fabric record."
  - "D15 (atomic write): settle whether a failed write is a reported skip, the order of the read-only check against the identical-content no-op, and whether the Bound covers the temporary file and its cleanup."
---

The D8 quoting point (runtime-expanded `#{socket_path}` and `#{client_name}` inside a single-quoted command string) is worth a look under the attack-surface question, with the bound it should state.
