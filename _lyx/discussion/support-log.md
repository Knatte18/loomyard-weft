# Support log — create-target-transient

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- A transient `Create` target marker in the plan format: it needs parser, validator and spec changes for a case one directory already identifies.
- A card validator rejecting a card that both creates and removes a path: the removal lives in the card's prose, which no mechanical check reads, and it would forbid the cleanup the scratch-file rule asks for.
- Exempting any path segment named `.scratch`: only the root `.scratch/` is git-ignored, so a nested one can hold tracked deliverables.
- Exporting a shared `.scratch` constant from `fabricengine` or `cleanupcli`: either import drags an engine into `planglyph` for one string.

## Review rounds

_No rounds yet._

## Question ledger

- Which remedy from the issue? Self-picked: exempt root `.scratch/` `Create` targets from `create-not-done`. Resolved.
- Do `Rename` new sides and `Delete` targets under `.scratch/` get the exemption? Self-picked: no, the report concerns only `Create`. Resolved.
