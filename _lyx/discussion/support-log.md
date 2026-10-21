# Support log — stencils-repo-neutral

## Interview

A darn run holds no interview: the writer read the board entry, the operator's direction in its imported issue comment and the shipped stencils, and made every call itself.

## Rejected alternatives

- Dropping the test-economy bullets from the rubrics entirely: rejected, since a redundant or missing test is a finding in any repository; only the LoomYard citation is repo-specific.
- Dropping the stencil-prose clauses as LoomYard-only: rejected, since they name no file or command and never fire in a hub without stencils, so they cause no search.
- A guard test scanning shipped stencils for `PATTERN-<id>`, with a PATTERN entry: rejected for this task, since `loom-plan-spec.md` documents a Go finding that names `PATTERN-fabric-vocabulary`, so the scan needs an exception until that gate is made repo-neutral.
- Rewording the `fabric-vocabulary-name` finding text in the spec: rejected, since the spec would then misdescribe what Go prints.

## Review rounds

_No rounds yet._

## Question ledger

- Should the test-economy rule stay in shipped rubrics? Self-picked: yes, without the citation.
- What replaces the CLI-shape bullet? Self-picked: a PATTERN-fit check against the hub's `PATTERN.md` and the background files the plan names, inactive without a `PATTERN.md`, per the operator's direction.
- Is the driver guide's `PATTERN-fabric-git` citation in scope? Self-picked: yes, the driver guide is shipped to every hub.
