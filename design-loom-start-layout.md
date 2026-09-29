Observed on the first real `lyx loom start` (go driver, task `burler-exclude-warn` in a loomyard hub), the task's tmux window holds four panes, top to bottom:

1. `lyx loom status --watch` — takes about a third of the window height. **Must be a one-line status strip**, as `docs/overview.md` already describes it ("a 1-line top pane").
2. `loom-operator`: an empty bash that `lyx loom start` adds (`operatorStrandAddSpec`, `internal/loomcli/bootstrap.go`, anchored `AnchorBelowParent`, so it lands directly under the status pane). **Must not exist.**
3. The producer agent (Discussion, then each later producer, each its own session) — the pane the work happens in. Gets only about a third of the window height; **must take all remaining height.**
4. **Selvage**, reed's permanent one-line shell band at the bottom (`internal/reedengine`, `selvagepane.go`). **This is exactly the operator terminal wanted** — keep it as is.

Wanted layout, top to bottom: one-line status, the agent pane filling everything else, Selvage.

## Why there are two operator shells

Both landed on 2026-09-19 from parallel work that was never reconciled: `d39b30648` replaced reed's header pane with a status line and Selvage, and `8ca2b9dbf` ("reed: born-as-strand for loom start's operator attach") added `loom-operator` as a separate tracked strand for the operator to attach in.
Selvage already is the operator's terminal, so `loom-operator` only duplicates it.

## Fix

- Stop adding `loom-operator`. If born-as-strand's tracking of the operator's pane is still needed (addressing, e.g. the mailbox design), make Selvage that tracked pane rather than adding another.
- Pin the status strand to one line.
- Size the agent pane to the rest.
- Check the same for the llm driver's layout, which adds a ly-drive strand.

Source: https://github.com/Knatte18/loomyard/issues/279 (close it in the landing commit).