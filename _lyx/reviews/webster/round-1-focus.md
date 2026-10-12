---
round: 1
focus:
  - "Top-strand placement in reed (card 2): does the first top strand split full-width above every pane and a later one below the lowest top strand, per the physical-top-placement decision, without relying on the layout string?"
  - "Title-row handling (cards 1 and 2): is the window's title-row answer read once per apply and attach and used for both layout and pins, per the title-row-once decision?"
  - "Content-rows report path (cards 3 and 5): does a pane report with `agentname.StrandNameEnv` and its own `TMUX_PANE`, and does `Engine.SetContentRows` reject a report from the wrong strand or pane and stay bounded under the zoom bracket?"
  - "Selvage in the Alt+Up/Down cycle and Alt+z (card 4): is the marker display-only and confined to the strands' window, with no change to other windows' cycling?"
  - "Pane poll backoff and minute-resolution times (cards 5, 7, 9): does rendered text change at most once a minute, and does the poll back off between changes?"
  - "Run window close (card 12) and notice log (card 8): does the window close only on a 0 attach or a removed pair, and is each notice appended once per process?"
  - "Test economy: new tests in the termwindow, reed, statuspane and batten packages against existing coverage, and any package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` without that file edited."
---

Focus follows the plan's shared decisions, since each is a cross-card invariant the per-card checks can miss.
