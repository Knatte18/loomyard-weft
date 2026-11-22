# Sampler session loads the hub's language skills before it edits code

The sampler session edits and commits code, but its spec carried no `Skills`.
It therefore ran without the language skills the repository's other writers load, and the code it wrote was poorer for it.

Now the sampler spec carries the hub's language skills, the same list the darn writer, webster, weaver and the fixers get.
The list comes from the hub's `language.yaml`, so a Go hub gets the Go skills and a C# hub the C# skills.
No sampler-specific list and no fixed language exists.

## What changes

- `samplerengine.SessionInputs` gains `LanguageSkills`, and `SessionSpec` sets it as the spec's `Skills`.
  The agent loads them after the sampler role's `agents.yaml` skills and before it writes code.
- `loomcli` wiring passes the `languageSkills` it already loads once for the hub into the sampler's `SessionInputs`.
- The parity producer for the sampler in `shuttleparity` builds its expected spec with the same skills.
- `spec_test.go` no longer pins that the sampler carries no skills; it pins that the skills passed in are carried.
- `TestWire_LanguageSkills` also wires a sampler seed over the same hub and asserts its skills equal the darn writer's.
  `writeSamplerConfig` moved from a closure in `TestWire_SamplerRecipe` to a shared helper for that.

Closes nothing on the issue tracker; the board entry lists no issue.

## Design decisions

- One source of language skills for every writer: the sampler reads the value `wire` already holds, not a second lookup.
- The test compares against the darn writer's skills rather than a literal, so the two cannot drift apart.

## To check by hand

Nothing outstanding; the change is covered by the tests above.

## Merge danger

**Door:** two-way
**Blast radius:** sampler
