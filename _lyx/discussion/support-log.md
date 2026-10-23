# Support log — prime-owner-visible

## Interview

A darn run holds no interview: the decisions come from the board entry and a reading of `internal/boardengine` and `internal/boardcli`.

## Rejected alternatives

- Adding the prime owner to `Outputs`: `Outputs` is config read from `board.yaml`, and the prime owner is board data.
- Changing `ListTasksBrief` to return the prime owner: it would touch every caller and test of a method whose other callers do not need it.
- A prime row in the README's Running table: the prime is not an entry and has no slug, recipe or status.
- Omitting the README and `--text` line when no prime owner is recorded: the absence then reads like an older binary instead of a stated fact.

## Review rounds

_No rounds yet._

## Question ledger

- Where in the README does the owner go? Self-pick: the opening paragraph, after the render time.
- Does `find --text` show it too? Self-pick: no; the board entry names `list`, and `find` is a search result.
- Is the JSON key omitted when empty? Self-pick: no, always present, empty when none is recorded.
