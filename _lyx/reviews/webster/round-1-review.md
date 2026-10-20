---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/burlerengine/doc.go:246
    summary: burlerengine doc still says ValidateFanKeys resolves every named fan through burler.yaml and places FanList and ValidateFanKeys in burlerengine
    fix: In the fan key grammar paragraph at internal/burlerengine/doc.go:245-246, name lenses.FanList and lenses.ValidateFanKeys, say they live in the lenses package, and replace "through burler.yaml" with "through lenses.yaml"; keep the ModelSpecList sentence, which stays in burlerengine.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: internal/lenses/fankeys.go:54
    summary: ValidateFanKeys doc says lenses.yaml falls back per name to the embedded template, which is burler-era wording; the load's fill now supplies the template entries
    fix: In the ValidateFanKeys doc comment at internal/lenses/fankeys.go:54, replace "which falls back per name to the embedded template" with "whose load fills a lens or fan the stack lacks from the template".
    origin: lens:code-affected-set
---

### [LOW] burlerengine doc still names burler.yaml for the fan key grammar

**Location:** internal/burlerengine/doc.go:246
**Issue:** Card 5 rewrote the neighbouring cluster fan-out paragraph to `lenses.yaml`, and card 8 retires burler's config, but the paragraph on `FanList` and `ValidateFanKeys` still says every named fan resolves "through burler.yaml".
That file is no longer a module or read by anything, so a reference to it remains after the retirement.
Both symbols now live in `internal/lenses`, so the paragraph also points at the wrong package.

### [LOW] ValidateFanKeys doc keeps the per-name template fallback wording

**Location:** internal/lenses/fankeys.go:54
**Issue:** The comment says `lenses.yaml` "falls back per name to the embedded template".
Card 3 gives `ResolveFan` no per-name fallback; `Load`'s fill supplies the template entries, and `Load` and `ResolveFan` in `config.go` are worded that way.
The sentence contradicts the sibling comments.

## Rejected

- Unrelated `--set` silently replacing a non-mapping per-name value (`internal/yamlengine/set.go`, code-correctness): the fork did not confirm that the value is dropped, and the card requires the refusal only for a pair under P.
- `keptExtra` in `hubreconcile` `logResult` has no test (code-tests): a log attribute, not a contract; card 4 lists no row for it.
- `TestResolveFan` unknown-fan rows overlap (code-tests NIT): the card itself renames the subtest, and the second row pins the known-fans text.
- `TestSet` per-name rows re-pin yamlengine writes (code-tests NIT): the fork's own analysis shows they pin the `PerNameMaps` pass-through no yamlengine test reaches.
- `internal/loomengine/config.go` header (code-pattern-fit F3): the fork states no defect, and the diff already moves every `burler.yaml` mention in that header to `lenses.yaml`.
- Declared PATTERN changes, per-name `kept` / `KeptExtra` never in `removed`, `PerNameMaps` mirror and fingerprint, burler deregistration and the two key moves: checked by the lenses and by the handler, all hold.
