```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to a goal point of the board entry or to the record's added entry.

**Decisions** (the `## Goal` points of the board entry, plus the record's `Added after Discussion` entry):
- G1: the sampler session loads the language skills of its repository before it edits code, the same way the darn writer gets them.
- G2: the skills follow from the hub's languages, never a fixed language.
- G3: `LanguageSkills` is wired into `SessionInputs` and onto the spec.
- G4: the test is changed to pin that the skills are carried.
- A1 (added entry): the sampler carries exactly the same skills as every writer, from the same source (the hub's `LanguageSkills`), with no sampler-specific list. A test pins that the sampler's skills equal the darn writer's.

**Changes and what they trace to:**
- C1, `internal/samplerengine/spec.go`: adds the `SessionInputs.LanguageSkills` field and sets `Skills: in.LanguageSkills`. Traces to G1, G3 and A1.
- C2, `internal/loomcli/wiring.go:1369`: passes `LanguageSkills: languageSkills`, the same `language.Skills` value loaded at `wiring.go:653` that the darn writer, webster and the fixers receive. Traces to G1, G2, G3 and A1.
  - The board text names `loomcli/samplerwiring.go` as the wiring site. The `SessionInputs` literal sits in the `wiring.go` closure that is handed to `samplerDeps`, and that closure is the only place the inputs are built. So the location differs from the text, but the substance matches. This is not a finding.
- C3, `internal/samplerengine/doc.go`: one sentence noting that the spec carries the hub's language skills. This is the doc update C1 forces.
- C4, `internal/samplerengine/spec_test.go`: changes the "no skills" assertion to assert that the skills given are carried. Traces to G4 ("change the test").
- C5, `internal/loomcli/wiring_test.go`, in `TestWire_LanguageSkills`: wires a sampler seed over the same hub and asserts its skills equal `chair.Skills`. Traces to A1's test requirement.
- C6, `internal/loomcli/hubfixture_helper_test.go` and `wiring_darn_test.go`: moves the `writeSamplerConfig` closure to a package-level helper. This is forced by C5, which needs the helper outside `TestWire_SamplerRecipe`. It is a behaviour-neutral move.
- C7, `internal/shuttleparity/producers_test.go` `buildSampler`: adds `Skills` from `languageSkillsOf(env)`. This is forced by C1: the parity producer mirrors the production spec, which now carries skills (PATTERN-engine-parity).

**Each decision and the changes that build it:**
- G1: C1 and C2.
- G2: C2. It draws from `language.yaml`, with no literal language.
- G3: C1 and C2.
- G4: C4.
- A1: C2 (one source) and C5 (the test against the darn writer).

**Findings:** none. Every change traces to a decision or is forced by a traced change, and every decision is built by at least one change.

Tool calls: none. My inherited context held the record, the diff pieces, the `wiring.go` closure and the `languageSkills` call sites.
