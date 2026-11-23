# Hardener focus-finding wiring is not tested end to end

`hardener-focus-wiring-untested` · Note · bug, loom

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `hardener-focus-permission-wiring-untested`.

The wiring that lets a hardener round raise a focus finding (`FocusFindings` into the round context) is not covered end to end, in `internal/burlercli` and `internal/shedadapters/roundcontext.go`.
