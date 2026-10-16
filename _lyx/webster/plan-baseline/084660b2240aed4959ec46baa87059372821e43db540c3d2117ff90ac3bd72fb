# Card 12 — webster-inline-pack-removal

**Delete:**
- `contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md`
- `contracts/stencils/webster/webster-fragment-locate-pack-inline.md`
- `contracts/stencils/webster/webster-fragment-pack-section-inline.md`
- `contracts/stencils#WebsterFragmentLocatePackInline`
- `contracts/stencils#WebsterFragmentFreshReadPackInline`
- `contracts/stencils#WebsterFragmentPackSectionInline`
- `internal/websterengine#PackLayoutInline`
- `internal/websterengine_test#dropInlineMarker`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#WebsterFragmentCardReadPack`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#TestStencils_MarkerPositions`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-fragment-locate-pack.md`
- `.gitattributes`
- `internal/websterengine#Config`
- `internal/websterengine#ConfigSpec`
- `internal/websterengine/template.yaml`
- `internal/websterengine#packLayoutOf`
- `internal/websterengine#renderLocate`
- `internal/websterengine#renderReadRules`
- `internal/websterengine#renderPackSection`
- `internal/websterengine#RenderForkPrompt`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#buildKickstartPack`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#writePrePackBody`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`
- `internal/websterengine_test#TestLoadConfig`
- `internal/configcli#TestDispatchLayers`

**Intent:**
D4: the inline kick-start pack layout goes, and `kickstart_pack_file`, which only picked between the file and inline layouts, is retired.

- The three inline fragments, their embeds and their `entries` rows, and their `.gitattributes` lines go; `WebsterFragmentCardReadPack`'s doc no longer names the inline layout.
- The implementer body loses its `{{.kickstart_pack_inline}}` marker; `RenderForkPrompt` and `RenderRecoveryPrompt` drop it from their value maps and optional lists, and `RenderForkPrompt`'s inline case and doc go.
  `PackLayoutInline` goes, and `renderLocate`, `renderReadRules` and `renderPackSection` drop their inline branches; `renderPackSection` keeps only the file layout's stencil, so its `sectionStencil` parameter stays for the one value.
- `Config` loses `KickstartPackFile`, and `ConfigSpec` moves `kickstart_pack_file` from `Categories` to `RetiredKeys` with `configengine.CategoryNone`, so a hub `webster.yaml` still holding it loads and reconcile prunes it.
  `packLayoutOf` returns `PackLayoutCard` when `KickstartPackPerCard` is set in a hub geometry, and `PackLayoutFile` otherwise; `KickstartPackPerCard`'s doc and `template.yaml`'s line for it drop the reference to `kickstart_pack_file`, and `template.yaml` loses the `kickstart_pack_file` line.
  A standalone geometry now always gets the file layout.
- `buildKickstartPack`'s doc lists `pack_layout` as `file` or `card`; `doc.go` drops the inline layout, the frozen-baseline (`1e39afc10^`) note and `kickstart_pack_file`, and says `kickstart_pack_per_card` picks between the per-card and file layouts.
- `webster-fragment-locate-pack.md`'s banner says it fills the file layout, picked when `kickstart_pack_per_card` is false or the geometry is standalone.

Tests:
`TestStencils_TopLevelMarkers` loses the three inline rows and `TestStencils_MarkerPositions` its inline-pack row.
`forkTemplateMarkerValues`, `writePrePackBody` and `TestTemplates_FillRequiresEveryRequiredMarker`'s optional lists drop `kickstart_pack_inline`; `dropInlineMarker` goes with the `TestRenderPrompts` rows that compare against the inline layout or render it.
`TestBeginBatch_KickstartPack` drops its `inline` field and the inline rows, keeps its file and per-card rows, and its per-card-off row asserts the file layout.
`TestLoadConfig` drops `KickstartPackFile` from its wants, its non-boolean `kickstart_pack_file` row becomes a row whose hub file still holds `kickstart_pack_file: false` and loads with the defaults, and its round-trip body drops the key.
`TestDispatchLayers`' non-boolean row sets `kickstart_pack_per_card=maybe` in place of the retired key.
The prune itself is configengine's generic handling of a `RetiredKeys` entry, which `internal/configengine/layers_test.go` already pins for a `CategoryNone` key; the `TestLoadConfig` row pins the webster half, a hub file still holding the key loading with the defaults.

**ImpactSummary:** the inline layout's three stencils, one layout constant and one config field go; the file and per-card layouts and every prompt without a pack render as before.
