# Support log — `lyx config --set` writes a per-round model list as a string

## Interview

A darn run holds no interview; the board entry `config-set-round-list` is the whole statement of the change.

## Rejected alternatives

- A new `configreg`/`configengine.Spec` field listing scalar-or-list keys: rejected, loom already declares exactly these keys as open maps, so the shape rule can key off that declaration.
- Choosing the written shape from the key's current node shape: rejected, it is today's behaviour and cannot move a key between a scalar and a list.
- Parsing every value as YAML: rejected, it would change how plain scalars with YAML-significant characters are written.
- A lyx-specific comma-split list grammar to accept `[opus[medium], sonnet[high]]` unquoted: rejected as a second list grammar for a handful of keys; the refusal names the quoted form instead.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Which keys get the scalar-or-list rule? Self-pick: every declared open-map path whose current value is not a mapping, which in loom is exactly the per-round model keys and `discussion_advisors`.
- Q: Accept the unquoted bracketed list? Self-pick: no; refuse with the quoted form as the way forward.
- Q: Validate model-spec grammar on write? Self-pick: no; loom's load-time validation and the local layer's `CheckValues` stay the checks.
