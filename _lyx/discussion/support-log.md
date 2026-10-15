# Support log — reed-launch-fixes

## Interview

A darn run holds no interview; the scope comes from the board entry, issues #627 and #628, and the prime's traces of the failed launches.

## Rejected alternatives

- **Make `Display.Focus` false mean "never take the viewer" everywhere.**
  Rejected: loom's driver, arbiter and burler spawns leave `Focus` false and rely on the follow rule so an attached viewer follows the newest agent.
- **Turn the shuttle `--focus` flag into a tri-state string.**
  Rejected: the bool's default is true, so an explicit false is already unambiguous.
- **Send a leading Enter instead of a discard key.**
  Rejected: it runs the stray bytes as a command, printing an error in the agent's pane.
- **Wait for the pane shell's prompt before typing.**
  Rejected: reed cannot read readiness from a default login shell without parsing prompts, and stray input can arrive before the prompt as well.
- **Send Escape on PowerShell.**
  Rejected: Escape followed by the typed `.` reads as Alt+`.`, which PSReadLine binds to YankLastArg.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Is the "outside the prime's session" flag or the `--anchor hidden` failure needed for the fix?
  Self-pick: no; both stay out of scope.
- Q: Does `lyx reed add --focus` change too?
  Self-pick: no; its default is false and means "follow", and the issue names only shuttle.
- Q: Where does the discard key live?
  Self-pick: `internal/shell`, beside the dialect's other line-level facts.
