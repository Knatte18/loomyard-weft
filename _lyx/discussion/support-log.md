# Support log — implementer-seam

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- A recipe-level `engine_setting:` field on a row, resolved by the recipe builder: it would teach `internal/shedbuild` about settings, where one registry entry does the same within the existing `Env.Setting` route.
- Validating `loom.implementer` in `loomengine.LoadConfig` as well: it would repeat the implementer table outside `internal/shedrecipe`, and the two lists could drift.
- Renaming the `Webster` row to an engine-neutral name now: it breaks resume and every Webster-Review name for a choice Weaver should make.
- Migrating a status file parked at `Batchifier` onto `Webster`: the recipe restarts runs on retired rows, and rewriting history would credit Webster with a Batchifier Done.
- Registering `implementer-spec.md` for deployment: no agent reads it at run time.

## Review rounds

_No rounds yet._

## Question ledger

- Which name does the setting take? Self-picked: `loom.implementer`, the board's example.
- Does the row keep the name `Webster`? Self-picked: yes, see the record.
- Does other Weaver groundwork, such as a cold-agent spawn per batch, belong here? Self-picked: no, it stays with Weaver.
