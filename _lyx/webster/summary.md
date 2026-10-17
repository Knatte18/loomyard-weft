# Creel CLI output fixes

Batch 01-log-id-reminded recorded `done` at head `6d1e80810afe422d6ef8194e8322be8af0124550`.

The batch fixed three `lyx creel` output bugs:

- `Store.Log`'s id filter also keeps events whose unread ids hold the id.
- `writeRefusal`'s stderr fallback prepends `creel: ` only when the message lacks it.
- A success envelope that cannot be written exits 1 with one stderr line naming what the verb already did.

Deviations from the digest: `internal/creelcli/cli_test.go`, changed outside the batch's own targets.
Departures: none.
