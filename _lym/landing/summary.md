# Sampler session loads the hub's language skills

The sampler agent edits and commits code, but its session spec carried no skills, so it started without the language skills every other writer loads.
`samplerengine.SessionSpec` now takes `SessionInputs.LanguageSkills` and puts it on the spec's `Skills`.
`loomcli`'s wire passes the hub's `language.yaml` skills, the same `languageSkills` value it hands the darn writer, webster and the fixers.
The agent loads them after the sampler role's `agents.yaml` skills, so a Go hub gets the Go skills and a C# hub the C# skills, with no list of its own.

## Tests

- `TestWire_LanguageSkills` wires a sampler seed over each hub it wires a darn seed over,
  and asserts the sampler session's skills equal the darn writer's.
  On the merge base the sampler spec carries no skills, so the row listing two Go skills fails there.
- `TestSessionSpec` now asserts the spec carries the skills it is given, in place of asserting it carries none.
- The engine-parity builder for `internal/samplerengine#SessionSpec` loads the hub's language skills, as the pr-reviewer's builder does.

## Merge danger

**Door:** two-way
**Blast radius:** sampler
