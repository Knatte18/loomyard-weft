```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or "none".

**Per-behaviour list**

| Behaviour | Pinning test |
|---|---|
| File render writes the shipped section first, then the repository's under its own heading (heading only when shipped text precedes it) | `render_test.go` `claudeFileWant` rows in `TestRender`, plus `ClaudeFileHoldsTheShippedLayerWithNoEntry`. Pinned. |
| Codex file target gets no shipped section | The existing codex row (`Marker + "\n\nrepository rules\n\ncodex text\n"`). Pinned. |
| Override replaces the shipped text only while `lym_override` is on, in Render | `OverrideReplacesTheShippedTextOnlyWhileTheSettingIsOn`, SettingOn and SettingOff rows. Pinned. |
| Same override in `Show` | None. `Show` is tested only with a repository entry (`render_test.go` Show case). |
| Same override in `Passages` | None. |
| Passages list the shipped layer's prompt entries first | None. `shipped.json` has no prompt-target entry, and the `Passages` tests use repository stores only. |
| `Set` and `Remove` refuse a shipped slug | `store_test.go` `ShippedSlug` row and `RemoveRefusesAShippedSlug`; CLI `OverrideVerbs` first two refusals. Pinned. |
| `SetOverride` refuses an unshipped slug and an empty text, leaving the store unchanged | `override_test.go` `SetRefusals`. Pinned. |
| `SetOverride` replaces rather than duplicates, and `RemoveOverride` drops and then refuses an absent override | `override_test.go` `SetReplacesAndRemoveDrops`. Pinned. |
| Overrides save to `heddle-lym-override.json` and load back; an absent file loads as empty | `SetReplacesAndRemoveDrops` for the round trip. The CLI `OverrideVerbs` first set starts from an absent file. Pinned. |
| CLI `--layer lym`: refused when the setting is off, unknown layer, `--order` or `--targets` given, missing `--text-file`, unshipped slug; no sync and no save on refusal | `heddlecli` `OverrideVerbs`. Pinned. |
| CLI `list` marks each entry's layer and each overridden entry, and `overridden` is absent when the setting is off | `TestHeddleCLI` list rows and `OverrideVerbs`. Pinned. |
| CLI `get` returns the override text | `OverrideVerbs` (setting on). Pinned. |
| `remove --layer lym` works with the setting off, and writes only the override file | `OverrideVerbs` (`rec.saves == 0`, two syncs). Pinned. |
| `heddle` config module is hub-wide and registered | `configreg_test.go` `HubWide` flags; `configstrictness_test.go` degrading set. Pinned. |
| `lym_override` defaults to off | Only indirectly. Every override test writes `heddle.yaml` explicitly; no test renders with an override file present and no `heddle.yaml`. |
| Local layer refuses `lym_override` (`CategoryCrossPair` in `ConfigSpec`) | None. |
| Batching sentence leaves the appended system prompt | `prepare_test.go` and `settings_test.go`, exact-equality assertions on the file content. Pinned. |
| Batching sentence is the shipped layer's first entry, aimed at `claude-file`, text unchanged, slug `lym-batching` | Not pinned by text. The tests derive expectations from `shippedStore`, so a changed or reworded `shipped.json` text still passes. A deleted entry makes `claudeFileWant` and `shippedSlugs()[0]` fail. |
| `decodeShipped` panics on a malformed, invalid or duplicate-slug shipped layer | None. |
| Malformed override file with the setting on makes `ReadLayers` fail | None. |
| Stale override (slug no longer shipped) is ignored in `Ordered` | None. |
| Repository entry sharing a shipped slug, already in `heddle.json`: renders and lists both, and `Remove` and `Set` refuse it | None. |
| The Bolt replayer does not claim `heddle-lym-override.json` | None (`replayer.go` `Claims` is unchanged; no test row for the new file). |

**Findings**

1. **MEDIUM, design** — `internal/heddleengine/render.go:281-325` (`Show`, `Passages`, `passagesOf`). The override and shipped-first ordering are changed in `Show` and `Passages` but not pinned. Every override test goes through `Render`. Breaking the override in either path, or the shipped-first order in `passagesOf`, passes every test. Evidence: `render_test.go` `OverrideReplacesTheShippedTextOnlyWhileTheSettingIsOn` calls only `mustRender`; the `Show` and `Passages` tests use repository-only stores. Fix: add rows to the existing `Show` and `Passages` tests. Use a test-only shipped entry aimed at `claude-prompt`, or a seam, since `shipped.json` has no prompt entry.

2. **LOW, design** — `internal/heddleengine/shipped.go:361-377` (`ReadLayers`). The failure paths have no test row. A malformed `heddle-lym-override.json` with the setting on, and an override whose slug is no longer shipped, are both untested. Evidence: `override_test.go` and `render_test.go` contain neither case. Fix: add one row each to `TestOverrides` or `TestRender`: a malformed file gives an error naming `git -C ... checkout`, and a stale slug is ignored.

3. **LOW, design** — `internal/heddleengine/shipped.go:309-325` (`decodeShipped`) and `internal/heddleengine/template.yaml`. Three behaviours are unpinned: the panic on an invalid or duplicate shipped entry, `lym_override` defaulting to off with no `heddle.yaml`, and the local layer refusing the key. Fix: one table-driven test calling `decodeShipped` with bad bytes and recovering, and one render row with an override file present and no `heddle.yaml`.

4. **LOW, design** — `internal/heddleengine/shipped.json:3-9`. The batching text and its `claude-file` target are not pinned. The tests compute expectations from `shippedStore`, so rewording the sentence or changing its target passes. The decision requires the text unchanged. Fix: one assertion in `store_test.go` or `render_test.go` that the first shipped entry has slug `lym-batching`, target `claude-file` and the exact sentence. This is a pin on behaviour, not stencil prose.

5. **LOW, design** — `internal/heddleengine/store.go:247-301` (`Set`, `Remove`, `refuseShipped`). When `heddle.json` already holds a repository entry under a slug that later ships, no test covers the result. Evidence: `store_test.go` `ShippedSlug` and `RemoveRefusesAShippedSlug` use a store holding the slug, so they pin the refusal. The render and list behaviour of such a collision has no row. Fix: add a collision row to `TestRender` or `TestHeddleCLI` that pins whatever the intended behaviour is.

6. **LOW, design** — `internal/heddleengine/replayer.go:24` (`Claims`) and `override.go`. The new Bolt file `heddle-lym-override.json` is written by verbs and committed by the Bolt sync. No test states whether the replayer claims it or leaves it unclaimed. Evidence: `replayer_test.go` is not in the diff. Fix: add a `Claims` row for `OverrideFileName`, with the expected value being whichever the design intends.

7. **NIT, consistency** — `internal/heddlecli/cli_integration_test.go` `OverrideVerbs` row `set house-rules --layer lym`. This duplicates `override_test.go` `SetRefusals/UnshippedSlug`. The CLI row still pins the verb's wiring, so the overlap is small. Fix: keep one if trimming.

**Tool calls:** none. The test diff was already in my context.
