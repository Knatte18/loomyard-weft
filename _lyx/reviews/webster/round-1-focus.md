---
round: 1
focus:
  - "Byte-identity: with kickstart_pack off, under a glyph-opt-out language, and in the recovery prompt, do the rendered texts (step 1, FRESH-READ, pack fragments) match the pre-change output exactly? Check the markers in webster-body-implementer.md and the pack-off fragments in contracts/stencils/webster/ against the previous text, and that a test pins it."
  - "Per-card grouping in internal/planglyph/pack.go, internal/planindex/planindex.go and internal/websterengine/beginbatch.go and render.go: is the pack resolved once per batch, grouped per card under its pointer path, and do the empty and unresolved cases render through the new pack-empty and pack-unresolved fragments?"
  - "Card 1: does a multi-line var or const signature reduce to its first line only, with func, type and other entries unchanged, and is the case covered?"
  - "Stencil registration: are the new fragments and markers registered in contracts/stencils/stencils.go and covered by claims_test.go and markers_test.go, and do the .gitattributes changes match the new files?"
  - "Test economy: do the new tests in template_test.go, beginbatch_test.go and pack_test.go duplicate coverage that already exists, and is any added behavior left uncovered?"
  - "Docs in the same commit: do internal/websterengine/doc.go and internal/planglyph/doc.go describe the per-card pack and the pack-on read, and does the pack still stay a kick-start with no source inlined?"
---

Seed focus only; the reviewer still reads the full range.
