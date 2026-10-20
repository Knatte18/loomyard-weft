---
round: 1
focus:
  - "Card 8 (retire-burler-module): check that no reference to burler's deleted config, its registration or its template is left, and that the two key moves migrate `burler.yaml` into `lenses.yaml` as the card declares."
  - "Cards 1, 2 and 4 (per-name maps): check that a name the template lacks is reported in `kept` / `Result.KeptExtra` and never in `removed`, at every layer, and that `PerNameMaps` is mirrored on `configreg.Module` and covered by the registry fingerprint."
  - "Cards 5, 6 and 7 (readers): check that every burler, loom and darn fan key and lens lookup resolves through `internal/lenses`, with no reader still on burler's old config."
  - "PATTERN changes for `PATTERN-leaf-packages` and `PATTERN-config-strictness`: check each declared entry lands with its declared text in the branch's `PATTERN.md` or `pattern/` diff."
  - "Test economy: check new tests in `internal/lenses` and the per-name map code against existing yamlengine, configengine and configsync tests for duplicated coverage, and check each added behavior has coverage."
---

Seed focus from the plan's card index and PATTERN changes; the review range follows the rubric's range section.
