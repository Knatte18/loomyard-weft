```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes on a concrete input.

**Per function or branch: the case I ran, and whether it is correct**

- **`ReinterpretJacquardBlocks`, parse and decode:**
  - An empty input gives Kind 0 and returns the input unchanged. Correct.
  - A comments-only document has no content and returns the input unchanged. Correct.
  - A top-level scalar or sequence fails `Decode` and returns a wrapped error, which the walk reports as a `FileError`. Correct; the file would not load anyway.
- **Profile loop:**
  - A shipped `solid` whose follow-added jacquard block equals the derived one takes the `DeepEqual` skip, with no line. Correct.
  - The `solid` row with a tuned tapestry block replaces the jacquard block in place and reports one rewrote line. Correct.
  - An operator-added `broad` whose only loom area is Discussion gets `jacquard: {drift:…}` with no loom module. Correct.
  - A profile whose tapestry block is `{loom: {discussion: …}}` and nothing else gets `jacquard: {}`. That loads, since `PrepareRunSettings` skips an empty set. Correct.
  - A profile with no tapestry block is skipped, and `holdsJacquard` keeps the decoded presence. Correct.
  - A profile value that is a scalar or null decodes to a nil profile and is skipped. Correct; the file would not load anyway.
- **Alias `quick: *b`:**
  - `base` is written into the anchored node first.
  - `quick` was decoded before the rewrite, so it differs; `ownMapping` follows the alias to the same node, and `setMappingValue` overwrites with an equal block.
  - The result is valid YAML with two lines. Correct.
- **Merge key inside a profile, `quick: {<<: *b, tapestry: T2}`:**
  - Its own mapping gets an own jacquard key built from T2, which overrides the merged `b.jacquard` built from T. Correct.
- **Profile supplied only through a root merge key:**
  - `ownMapping` returns nil, so the profile is flagged and `holdsJacquard` stays false. Correct.
- **Jacquard block that is an alias, shared between profiles (finding 1):**
  - Case: `solid.jacquard: &j {…}` and `wide.jacquard: *j`, where `solid`'s derived block differs and `wide`'s matches.
  - `setMappingValue` replaces `solid`'s anchored node, while `wide` keeps `*j`.
  - The encoder then writes an alias whose anchor is no longer in the document, so the output does not parse. **Defect.**
- **`jacquardSettings`:**
  - A tapestry block with no loom module is returned as a clone. Correct.
  - `loom: {}` drops the loom module. Correct.
  - `loom` holding Discussion and Code keeps Code. Correct.
  - The tapestry map is not mutated, since the loom map is cloned before the delete. Correct.
- **Values encoded with `block.Encode(want)` (finding 2):**
  - An int, a string such as `"off"` and the empty string `""` round-trip. Correct.
  - A whole-valued float such as `2.0` is encoded as `2`: yaml.v3's `floatv` formats with `'g'`, so it reads back as an int. Wrong type, though no shipped key is a float.
- **`reinterpretJacquardDefault`:**
  - `defaults.tapestry: light`, where `light` holds a jacquard block and `defaults.jacquard` is `solid`, appends or overwrites to `light`. Correct.
  - `defaults.jacquard` already equal returns no line. Correct.
  - With no `defaults.tapestry`, a held `defaults.jacquard` is dropped. Correct.
  - With no `defaults.tapestry` and no `defaults.jacquard`, nothing happens. Correct.
  - A null or absent defaults entry leaves `entries` nil and `named` and `held` false, so nothing happens. Correct.
  - Defaults supplied through a root merge key give a nil node, and both the set and the drop path return the merge flag. Correct.
  - A `defaults.tapestry` naming a missing profile reads false in `holdsJacquard`, so the default is dropped. Correct; the file would not load anyway.
- **Defaults mapping holding `jacquard` both as its own key and through a merge key (finding 3):**
  - `removeMappingKey` removes the own key and returns true, so the line reports "dropped".
  - The merged value still supplies `defaults.jacquard`, so the reported line is untrue and the default can still name a profile without a jacquard block.
- **Return paths:**
  - When only flagged lines exist, the input comes back unchanged with its flags. Correct.
  - `encodeDocument` errors propagate. Correct.
  - When nothing differs, the input comes back byte for byte. Correct.
- **`encodeDocument` (moved from the arras code):** same encoder and indent, and the arras path calls it with `&document`. Correct; no behaviour change.
- **`ownMapping`:**
  - A missing key returns nil.
  - An alias to a mapping returns the anchored mapping.
  - An alias to a scalar, or a null value, returns nil.
  - Correct.
- **`setMappingValue`:** writes over an own key in place, and appends when the key is absent. Correct.
- **Template `jacquard` blocks:** each equals its profile's tapestry block minus `loom.discussion`, including `speed`'s `reviewer_fan: ""`. Correct.

**Findings**

1. LOW, design — `internal/runprofile/reinterpret.go:249` (`setMappingValue` replaces the value node).
   Replacing an anchored jacquard node that another profile's jacquard aliases leaves a dangling alias, and the rewritten `profiles.yaml` does not parse.
   No check catches it: `internal/hubreconcile/reinterpret.go:68-73` writes the bytes with no load check.
   Reaching it needs a hand-written anchored jacquard block before the one-time run, so it is unlikely.
   Evidence: `reinterpret.go:122-128` replaces with a freshly encoded node, and `ReinterpretJacquardBlocks` never checks for aliases pointing into the node it replaces.
2. NIT, design — `internal/runprofile/reinterpret.go:124-125`.
   The derived block is re-encoded from decoded Go values, so a whole-valued float in a tuned tapestry block, such as `2.0`, comes back as the int `2` in the jacquard block.
   Evidence: `floatv` in yaml.v3 v3.0.1 `encode.go:398` formats with `'g'` and emits a plain, untagged scalar.
   Copying the tapestry block's own nodes, minus Discussion, would keep the types.
3. NIT, design — `internal/runprofile/reinterpret.go:177-182`.
   When the defaults mapping holds `jacquard` both as its own key and through a merge key, the drop removes only the own key.
   The merged value survives, so the "dropped" line is untrue and the default may still name a profile without a jacquard block.
   Evidence: `removeMappingKey`, `reinterpret.go:258-270`, only reports whether an own key was removed.

No other wrong result, swallowed error or inconsistent failure-path state was found in the changed code.

Tool calls:
1. grep of the module templates for float values, and the yaml.v3 `floatv` source: my context lacked how yaml.v3 encodes a whole-valued float and whether any block key is a float.
