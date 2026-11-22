```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that fails when it breaks.

**Behaviours and the tests that pin them:**

1. **`samplerengine.SessionSpec` puts `in.LanguageSkills` on `Spec.Skills`** (`internal/samplerengine/spec.go:96`).
   Pinned by `TestSessionSpec` (`internal/samplerengine/spec_test.go:40,61`).
   The input carries two Go skills and the assertion is `slices.Equal(spec.Skills, valid.LanguageSkills)`, so dropping or reordering the field fails.
   At the merge base the assertion read `len(spec.Skills) != 0`.
2. **The `loomcli` wire passes the hub's `languageSkills` into the sampler `SessionInputs`** (`internal/loomcli/wiring.go:1369`).
   Pinned by `TestWire_LanguageSkills`, row "a listed language reaches the Webster deps and the darn writer…" (`internal/loomcli/wiring_test.go:1289-1303`).
   That row compares the sampler spec's skills to `chair.Skills`, and the row has already checked `chair.Skills` against the two-skill literal `wantSkills`.
   A missing line gives nil against two skills, so the row fails.
   The value comes from the same `language.yaml` load at `wiring.go:653`.
   - The `skills: []` row only checks that an empty list stays empty.
     A wiring break would pass that row, but the Go-skills row already covers the break, so this is no gap.
   - The rows with an unusable `language.yaml` return before reaching the sampler, so nothing checks that a sampler seed refuses a bad `language.yaml`.
     That refusal runs through the shared load at `wiring.go:653`, which comes before the recipe-specific branches, and those rows already pin it.
     The diff adds no sampler-specific refusal, so this is no gap.
3. **The engine-parity producer `buildSampler` carries the language skills** (`internal/shuttleparity/producers_test.go:233-252`).
   This is test-support code, not a production behaviour.
   It feeds the parity test, which expands `Agents.Skills(ConfigRole, spec.Skills)` (`producers_test.go:516`).
   The fixture seeds `language.yaml` with `scribe:prose` (`internal/shuttleparity/seats_test.go:79`), so the skill reaches both engines' manifests and the comparison has something to compare.
   It follows the same pattern as `buildPRReview`, `buildMerriam` and `buildRecovery`.
4. **`writeSamplerConfig` moves from inside `TestWire_SamplerRecipe` to a package helper** (`internal/loomcli/hubfixture_helper_test.go:130-137`).
   This is a test-only refactor with the same body and signature, so there is no new behaviour to pin.
   `TestWire_SamplerRecipe` still calls it with the same arguments.

**Findings:** none.
Every production behaviour the diff adds has a row that fails when it breaks.

**Tool calls:**
- One grep in `internal/shuttleparity` for its `language.yaml` seeding, to see whether the parity fixture gives `buildSampler` any language skills; the manifest did not include the parity fixture's config.
