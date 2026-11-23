# Decision accept below the soft-cap window passes an open BLOCKING finding

`below-window-accept-blocking` · Note · bug, loom

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `below-window-decision-accept-passes-open-blocking`.

A decision accept recorded below the soft-cap window passes over an open BLOCKING ledger entry of another class, in `internal/shedadapters/bouncerdecision.go`.
That lets a run land past an open BLOCKING finding, which the landed task was meant to stop: the run should ask the parent instead.
