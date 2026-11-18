# Card 6 — stencil-plain-includes

**Edit:**
- `internal/stencilstore#ReadForEngine`
- `internal/stencilstore#writeEngineBlocksInPlace`
- `internal/stencilstore#TestReadForEngine`
- `//internal/stencilstore/doc.go`
- `internal/landingshed#RenderPRReviewPrompt`
- `internal/samplerengine#RenderSessionPrompt`

**Uses:**
- `internal/stencilstore#readInclude`
- `//internal/samplerengine/doc.go`

**Intent:** Let every brief of a code-committing session include a plain fragment, which D2's one shared fragment needs (card 11).

Today `ReadForEngine` writes only engine blocks in place and leaves a plain `{{template "<name>"}}` as it stands, so `stencil.Fill` fails on it.
No stencil read through `ReadForEngine` carries a plain include today; only seat stencils and `burler-step-fix` resolve them, through `ReadSet`.

- `writeEngineBlocksInPlace` also writes each bare plain include in place: the stencil of that name read through `readInclude`, its own engine blocks written in place and its leading comment dropped.
  An include of a stencil that does not exist is the read's error, wrapped to name the including stencil and the missing one, with `way forward: run lym stencil diff, then lym stencil sync to seed the missing stencil`.
- `ReadForEngine`'s doc says so; `doc.go` drops the sentence that a plain include is left as it stands.
- `RenderPRReviewPrompt` and `samplerengine`'s `RenderSessionPrompt` read their stencil with `ReadForEngine` for the engine they already take, in place of `Read`, so their briefs can include the fragment.
- `TestReadForEngine`, tier0, gains a row whose stencil includes a plain fragment that itself includes an engine block, asserting the exact text with both written in place, and a row naming a missing fragment, asserting the error names both stencils and the way forward.

**ImpactSummary:** One reader gains plain-include resolution and two prompt renders switch to it; no shipped stencil read through it carries a plain include before card 11.

**Commit:** `6: stencil-plain-includes`
