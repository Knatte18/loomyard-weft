# Sampler session loads the language skills before it edits code

`sampler-loads-language-skills` · Note · bug, loom, shuttle

## What was seen (operator, 2026-11-22)

The sampler session of `readme-declutter` edited Go code without loading the Go language skills first, and the code it wrote was poor for it.

## Cause

`samplerengine.SessionSpec` (`internal/samplerengine/spec.go`) builds the sampler spec with no `Skills`, while darn, webster, weaver and the fixers pass the hub's `LanguageSkills`.
`spec_test.go` pins that the sampler spec carries no skills.

## Fix

The sampler session loads the hub's language skills before it edits code, the same way the darn writer gets them:
wire `LanguageSkills` from `loomcli/samplerwiring.go` into `SessionInputs` and onto the spec, and change the test to pin that the skills are carried.
