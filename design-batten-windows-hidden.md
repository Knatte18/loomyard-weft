# Batten windows stay but are hidden from the status bar

`batten-windows-hidden` · Task · enhancement, batten, reed · running · Darn-Write

## Seen

`lyx batten run --window` gives each batten run its own window in the prime's reed session, which is where its process lives, and each shows up as a `batten:<slug>` button on status line 1 of the orch's session.
Their output says nothing useful to the operator.

## Wanted (operator, 2026-10-08)

- The batten windows stay, since the batten process lives there, but they are not visible: they carry a marker the status bar skips, so status line 1 shows no `batten:<slug>` buttons.
- The runs overview (note `runs-overview`) is where runs are watched; until it exists, `lyx batten status` and each run's own terminal window cover it.
