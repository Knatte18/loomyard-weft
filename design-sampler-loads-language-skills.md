# Sampler session loads the language skills before it edits code

`sampler-loads-language-skills` · Task · bug, loom, shuttle · starting · Seed-IDE

## What was seen (operator, 2026-11-22)

The sampler session of `readme-declutter` edited code without loading the language skills first, and the code it wrote was poor for it.

## Cause

`samplerengine.SessionSpec` (`internal/samplerengine/spec.go`) builds the sampler spec with no `Skills`, while darn, webster, weaver and the fixers pass the hub's `LanguageSkills`.
`spec_test.go` pins that the sampler spec carries no skills.

## Fix

The sampler session loads the language skills of the repository it works in before it edits code, the same way the darn writer gets them.
Which skills follow from the hub's languages, never a fixed language: a Go hub gets the Go skills, a C# hub the C# skills (operator, 2026-11-22).
Wire `LanguageSkills` from `loomcli/samplerwiring.go` into `SessionInputs` and onto the spec, and change the test to pin that the skills are carried.
