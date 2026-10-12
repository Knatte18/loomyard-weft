---
round: 1
focus:
  - "Reed window options: check that `@lyx_own` and `@lyx_group` are set by window id, that reconcile ends only a window carrying `@lyx_own` with no bound pane, and that groups are found by `@lyx_group`, never by name; settle whether `@lyx_unseen` and `@strand_color` stay display-only."
  - "Window 0 invariance: check that `listStrandPanes` still lists window 0 only and that own-window panes join only the liveness reads through `presentPanes`; settle whether any layout, Selvage, split or reap decision now sees an own-window pane."
  - "Group sizing and split: check `window_share_max` and `window_split` handling in reed config, group assignment and relayout; settle the empty `window_split` reading as `width`, the validation at up, resume and add, and the relayout on removal."
  - "Removal paths: check that removing an own-window strand moves its viewer back to window 0, and that shuttle's stop paths remove dead own-window strands while keeping a died run's last capture; settle whether any path leaves an orphan window or loses the capture."
  - "`lyx shuttle open` and the provider seam: check that a named session opens with no prompt and no turn, in its own window, and that `OrElse` and `Exit` in the pane shell seam quote and compose correctly; settle error handling when the session name is missing or already live."
  - "Per-card mechanical checks and test economy: confirm each card's own group checks actually ran against every target, and that new tests in packages near their row in `cmd/lyx/testdata/test-budget.yaml` do not duplicate existing coverage while each added behavior has some."
---

Focus is derived from the plan's Shared Decisions and the card index; no prior round exists.
