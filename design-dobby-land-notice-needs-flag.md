# Dobby posts the can-land notice only once the beam is flagged ready

`dobby-land-notice-needs-flag` · Task · bug, dobby, workspaces · draft

## Seen (operator, 2026-10-30)

A beam was created under the Models hub.
Once its orch was up, the beam's dobby told the orch that nothing runs under the beam and asked it to run `lym beam land <slug>`, though the beam had only just been made and had no work yet.
The notice is `landNotice` in `internal/dobbyengine/landnotice.go`: it goes out whenever the landing's refusal finds nothing working under the beam, so an empty new beam reads as a finished one.

## Design (operator, 2026-10-30)

- The flag says the beam is ready to be merged; it does not say the project is finished, and dobby starts the landing and the teardown as soon as it is set.
- Dobby waits with the can-land notice until the flag is set.
  Without it, nothing working under the beam is no cause for a notice, however long the beam has been idle.
- The flag is a record in the beam's `_lym`, beside the other information about the merge, such as the describe field, and not a field on the board.
- `lym beam ready <slug>` sets it.
  The beam's orch runs it, or the operator; which of them does not matter.
  Dobby never sets it.
- `lym beam land` refuses without the flag.
