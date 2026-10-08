---
round: 6
focus:
  - "D1 (zoom handling) and D7 (top pin, row 0): decide whether the pane list and its physical order are read on the unzoomed window or from a source that does not change under zoom, including in the attach path where `AttachArgv` in `internal/reedengine/attach.go` lists panes before attach. A zoomed pane reports `pane_top` 0, and `paneIDsByTop` in `internal/reedengine/apply.go` sorts by `pane_top`. Settle whether the acceptance criteria zoom a strand that is not the top pane."
  - "D1 re-zoom trigger: settle whether it covers every reed step that changes the strand window's panes, layout or active pane, given that `split-window` without `-Z`, `kill-pane` of another pane and `select-pane -t` without `-Z` also unzoom, and whether the Bound's reading of zoom at step start closes the gap."
  - "D1 and D7 mechanism for executions reed does not run as Go steps: the attach chain is an argv tmux runs after `AttachArgv` returns, and each `window-resized` hook entry is a single tmux command. Settle whether a Decision states how these carry the unzoom, resize, re-zoom and the post-apply pin run."
  - "D12 booter re-run and Scope: settle whether the booter's re-run of its session-ensuring step carries the same skip as a revive function's up, so the revival trigger is not evaluated twice, and whether the Scope line and the Bound state the same revival predicate as the Decision."
  - "D15 temporary file: settle what happens when a file of the chosen temporary name already exists and whether cleanup removes only a file this run created."
  - "Constraints bullet under `PATTERN-told-geometry`: settle whether it states exactly what the hub-mode teller tells `reedengine` after D12's sort."
---

Entries restate the sites they depend on; none caps severity or pre-states a verdict.
