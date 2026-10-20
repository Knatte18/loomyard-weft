# Support log — lenses-template-frozen

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Byte or node-style equality for the drop: rejected, since a lens rewritten from a block scalar to a quoted string with the same text is not a tuning.
- Dropping template-equal names at the sparse layers through `Prune`: rejected, since a sparse name equal to the template can override a tuned value below and dropping it would change the loaded value.
- Leaving an absent per-name map absent in the merged file: rejected, since a full base layer holds every template key and a null there fails the load's fill.
- Leaving `SetValues` unchanged: rejected, since it builds from the template and would write every template name back into the file on any `--set`.
- Keeping the load's fill log for per-name names: rejected as per-load noise once missing names are the normal state.

## Review rounds

_No rounds yet._

## Question ledger

- Q: What counts as equal to the template? Resolved (self-pick): the decoded value, deep-equal.
- Q: Does the drop reach `--set`? Resolved (self-pick): yes, `SetValues` drops template-equal names after its pairs apply.
- Q: How does an absent or null per-name map reconcile? Resolved (self-pick): written as an empty mapping, P reported added.
- Q: Where are drops reported? Resolved (self-pick): `dropped` from `Reconcile`, `Result.Dropped`, the CLI's `dropped` and the hub walk's log.
