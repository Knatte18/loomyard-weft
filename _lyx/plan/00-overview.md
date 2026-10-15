---
format: 5
approved: true
language: go
---

# Plan: webster kick-start pack layout switch

A `webster.yaml` key, `kickstart_pack_file` (default `true`), picks how a Webster implementer fork prompt delivers its kick-start pack: today's pack-file layout, or the inline layout of `1e39afc10^` that the 2026-10-13 glyph audit measured.
The inline texts live in three frozen stencil fragments, the implementer body gains a second optional pack marker, `RenderForkPrompt` takes the layout, and each begin-batch's `webster: kickstart pack` trace line records `pack_layout`.

## Card Index

1 — config-key — add the `kickstart_pack_file` bool to webster's config, template and spec, defaulting to `true`
2 — inline-fragments — add and register the three frozen inline-layout fragments with the `1e39afc10^` text
3 — render-layout — the body's `kickstart_pack_inline` marker, `RenderForkPrompt` rendering either layout, begin-batch passing the key
4 — trace-layout — the `webster: kickstart pack` line gains `pack_layout` last

## Shared Decisions

- The inline fragments are the measurement baseline: their text is the `1e39afc10^` text and never follows an edit to the file-layout fragments.
- Pack off, language-none, an empty pack and the recovery prompt render today's bytes under either value of the key.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/websterengine/... ./contracts/stencils/... ./internal/configcli/... ./internal/configreg/...
go vet -tags tmux ./internal/websterengine/... ./contracts/stencils/... ./internal/configcli/... ./internal/configreg/...
go vet -tags llm ./internal/websterengine/... ./contracts/stencils/... ./internal/configcli/... ./internal/configreg/...
go test ./internal/websterengine/... ./contracts/stencils/... ./internal/configcli/... ./internal/configreg/...
go test -tags integration ./internal/websterengine/... ./contracts/stencils/... ./internal/configcli/... ./internal/configreg/...
```
