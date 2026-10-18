# Support log — creel-hold-log-spam

## Interview

A darn run holds no interview; the scope is read from the board entry and the code.

## Rejected alternatives

- Journal no renewal and keep the extended expiry in memory only: a daemon restart would revert the hold to its first expiry, and the hold could lapse during an orch cycle.
- Journal a renewal only when the persisted expiry nears: it bounds journal growth but still logs an event per write, and adds a threshold to tune.
- Keep `held` records with a renewal flag field: an older binary would still log every renewal, and the fold would branch on a field instead of the event name.
- Filter repeated `held` events at log read time: the journal still carries them as log events, and every log consumer would need the same filter.

## Review rounds

_No rounds yet._

## Question ledger

- Should a renewal stay durable? Self-picked yes: the journal is the daemon's only state, and a restart must keep the hold.
- Does an extended expiry count as a change the log shows? Self-picked no: a watcher extends it every tick, which is the spam the issue reports.
