---
round: 1
focus:
  - "contracts/stencils/webster/*-inline.md: settle whether each inline fragment's text is byte-identical to the `1e39afc10^` text (Shared Decisions), and whether the edits to the file-layout fragments (locate-pack, pack-card, pack-empty, pack-unresolved) leave today's bytes unchanged."
  - "internal/websterengine/render.go: settle whether pack off, language-none, an empty pack and the recovery prompt render the same bytes under both values of `kickstart_pack_file`."
  - "internal/websterengine/render.go and webster-body-implementer.md: settle whether the second optional marker `kickstart_pack_inline` is handled symmetrically with the existing pack marker, including when the body lacks one of them."
  - "internal/websterengine/beginbatch.go: settle whether begin-batch passes the config key to `RenderForkPrompt` on every call path, and whether the `pack_layout` field is last on the `webster: kickstart pack` trace line and matches the layout rendered."
  - "internal/websterengine/config.go, template.yaml, configcli: settle whether the key defaults to `true`, is registered in the template and spec, and whether doc.go documents the new layout and key."
  - "Test files (template_test.go, beginbatch_test.go, claims_test.go, markers_test.go): settle whether any new test duplicates an existing test's coverage, and whether any behavior the diff adds has no coverage."
---

Seed focus drawn from the plan's cards and Shared Decisions; each entry is a question to settle, not a verdict.
