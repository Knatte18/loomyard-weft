<!-- This is the kick-start pack section of webster's fork prompt.
     renderPackSection (render.go) reads it through stencilstore, fills its two markers with Go-rendered lists, and strips this banner and trailing newline;
     RenderForkPrompt fills the result into the optional kickstart_pack marker of webster-body-implementer.
     pack_entries lists the member glyphs the batch's cards name, each with its declarations;
     pack_unresolved lists the refs that did not resolve, each with its reason.
lyx-stencil: sha256=ba7d75ac6474f2b9c9bf3fa318b92283a68ad0d1e709e6ca5afcc6f6b6e245c1 build=c279a38bafb5aced5978bceb727ff9c06e756848 time=2026-10-11T15:39:39Z -->

## Kick-start pack

The member glyphs on this batch's cards, resolved against the tree at the batch's start.
Each entry gives the glyph, then one line per declaration with its file, line span and signature.

{{.pack_entries}}

### Unresolved

These refs on this batch's cards did not resolve to a declaration, each with its reason.

{{.pack_unresolved}}
