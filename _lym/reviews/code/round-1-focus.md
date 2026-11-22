---
round: 1
focus:
  - "internal/loomcli/wiring.go sampler closure (~line 1369): does the sampler seed get the same `languageSkills` value the darn writer, webster and the fixers read, from the one language.yaml load, with no sampler-specific list or fixed language?"
  - "internal/samplerengine/spec.go `SessionSpec`: do `in.LanguageSkills` reach `Spec.Skills` correctly, and does the new `LanguageSkills` doc comment ('after the sampler role's agents.yaml skills') match how shuttle and agentconfig order skills?"
  - "TestWire_LanguageSkills in internal/loomcli/wiring_test.go: does the sampler assertion compare against the darn writer's skills per case, as the decision record's added entry requires, and does it still cover the empty and unusable language.yaml cases?"
  - "internal/shuttleparity/producers_test.go `buildSampler`: does the parity producer now derive skills from the same source as production, so the parity check cannot pass vacuously?"
  - "Commit message of 2f607a494: does its `Root-Cause:` trailer name the defect the diff removes, and are the diff's helper moves (`writeSamplerConfig`) license-free refactors that leave `TestWire_SamplerRecipe` unchanged in behavior?"
---

Diff is small and confined to the sampler wiring, spec, and their tests.
The decision record asks for one source of language skills and a test pinning that the sampler carries the darn writer's skills.
