## Goal

```markdown
## What was seen (operator, 2026-11-22)

The sampler session of `readme-declutter` edited code without loading the language skills first, and the code it wrote was poor for it.

## Cause

`samplerengine.SessionSpec` (`internal/samplerengine/spec.go`) builds the sampler spec with no `Skills`, while darn, webster, weaver and the fixers pass the hub's `LanguageSkills`.
`spec_test.go` pins that the sampler spec carries no skills.

## Fix

The sampler session loads the language skills of the repository it works in before it edits code, the same way the darn writer gets them.
Which skills follow from the hub's languages, never a fixed language: a Go hub gets the Go skills, a C# hub the C# skills (operator, 2026-11-22).
Wire `LanguageSkills` from `loomcli/samplerwiring.go` into `SessionInputs` and onto the spec, and change the test to pin that the skills are carried.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-22): Sampler carries the same language skills as every writer

Decision: The sampler session loads exactly the same language skills as every other writer in the repository (darn writer, webster, weaver, fixers), from the same source, the hub's LanguageSkills; no sampler-specific list and no fixed language. The test pins that the sampler carries the same skills as the darn writer.

Rationale: Operator, 2026-11-22: one source of language skills for every writer in the repo.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
