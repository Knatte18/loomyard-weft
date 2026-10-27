# Support log — docs-link-skip-bench-data

## Interview

A darn run holds no interview.
The board entry's Fix section, written by the operator, is the full statement of the change.

## Rejected alternatives

- A `SkipDirs` or path-predicate option on `scankit.Options`: rejected because only this guard needs it, and a shared option for one caller is a premature abstraction.
- Skipping every `data/` directory anywhere under `docs/`: rejected because the entry names only the data of research reports, and a wider skip would hide broken links in other docs.
- Allowlisting each broken link in the captures: rejected because the captures grow with every bench and their links are never rewritten, so the allowlist would grow without end.
- Rewriting the captured links to resolve: rejected because the data is verbatim by design.

## Review rounds

_No rounds yet._

## Question ledger

- Is the `data/` directory the report directory's direct child only, or any `data/` below it? Self-pick: the direct child, which matches every report and the entry's wording.
- Where does PATTERN state the guard's scope? Resolved: nowhere in `PATTERN.md` or `pattern/`, so only the guard's doc comment changes.
- Does the skip also exempt links into `data/` files from the target check? Self-pick: no, the skip is source-side only.
