# A beam session shows the pawl run checklist in a second overview window beside the beam overview

`pawl-checklist-hidden-in-beam` · Note · enhancement, pawl, orch, reed

## Seen (operator, 2026-11-03)

A beam's pawl run is a shed run with its own checklist, the loom status strand a normal loom run shows in its session's overview window.
Once the beam is up, its session shows the beam overview, its orch and the shell, and the pawl run's checklist is nowhere, so the operator cannot see where pawl stands; that matters once pawl is advanced by hand.

## Cause, from the code

- At `lym beam start`, the pawl run's `lym loom start` puts the loom status strand into the session's one overview window.
- `ensureRunsStrand` (`internal/orchcli/runs.go`) then puts the beam's runs strand in that window by replacing the loom status strand, since reed refuses a second overview strand; the checklist strand is gone, not moved.
- `statusStrandAddSpec` (`internal/loomcli/bootstrap.go`) does give the checklist a window of its own beside the runs strand, but only when a later loom start or resume adds it, and that window keeps the viewer, so it is easy to miss.

## Expected

While a pawl run lives, the beam's session always shows its checklist in a window of its own, named for pawl, beside the beam overview:

- When the runs strand takes the overview window from the loom status strand, the checklist is added again in its own window, or the beam start brings the runs strand up before the pawl run's loom start so the checklist never takes the overview window.
- The beam overview may also carry one line with pawl's current row, pointing at that window.

## Two overview windows (operator, 2026-11-03)

The operator asks whether a session can show two overview windows.
reed's overview anchor today places one strand per session, alone in a window at the lowest index, and `AddStrand` refuses a second (`internal/reedengine/doc.go`, "Overview anchor").
The orch's recommendation: let the anchor hold a second strand in a beam's session, so the beam overview opens first and the pawl checklist second, both with the overview window's plain style and focus rules, and neither replaces the other.
A split of the one overview window into two panes was weighed and passed over, since both panes get narrow on a phone.
