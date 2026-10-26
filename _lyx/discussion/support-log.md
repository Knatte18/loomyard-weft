# Support log — pattern-line-cap-text-only

## Interview

A darn run holds no interview; the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Keep the whole-line cap and print the room left for the text in the finding: the number a writer works against would still vary per entry.
- Let `internal/discussionparser` import `internal/pattern` for the cap and extraction: the leaf allowlist admits no import, and `internal/pattern` pulls in stencil packages.
- Rename the checks to `pattern-entry-text-cap` and `long-text`: the names are stable identifiers in specs and run records, and the rename changes no behavior.
- A cap of 300 or 320: looser than the tightest room any current entry had, with no entry needing it.

## Review rounds

_No rounds yet._

## Question ledger

- Which cap value? Self-picked 280, the old room of the longest-named entry with a link; every current text is below it.
- Does the discussion bullet text carry `(test)` and the link? Not by contract, so the check strips them when present and measures the rest.
