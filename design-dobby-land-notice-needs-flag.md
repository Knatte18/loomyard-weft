# Dobby posts the can-land notice only once the beam is flagged ready

`dobby-land-notice-needs-flag` · Note · bug, dobby, workspaces

## Seen (operator, 2026-10-30)

A beam was created under the Models hub.
Once its orch was up, the beam's dobby told the orch that nothing runs under the beam and asked it to run `lym beam land <slug>`, though the beam had only just been made and had no work yet.
The notice is `landNotice` in `internal/dobbyengine/landnotice.go`: it goes out whenever the landing's refusal finds nothing working under the beam, so an empty new beam reads as a finished one.

## Design (operator, 2026-10-30)

- Dobby waits with the can-land notice until the beam has a flag saying it is considered finished and ready to merge.
- Without the flag, nothing working under the beam is no cause for a notice, however long the beam has been idle.
- The beam's orch sets the flag, or the operator; which of them does not matter.
  Dobby never sets it.
- `lym beam land` refuses without the flag.

## Open

- Where the flag lives: a field on the beam's project entry, set by a verb of its own, or a record in the beam's `_lym`.
