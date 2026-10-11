# Card 1 — darn-board-get-json

**Edit:**
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils#wordingClaims`
- `internal/darnengine#TestDarnSpec`

**Uses:**
- `contracts/stencils/loom/loom-template-discussion.md`

**Intent:** Decision D1.
`lyx board get` takes a JSON object, so the bare `lyx board get {{.slug}}` both stencils carry fails with `invalid json`.
Rewrite the darn writer's Step 1 line and the landing describe stencil's "The board entry, via ..." bullet to the form the discussion stencil already uses, `lyx board get '{"slug":"{{.slug}}"}'`, and nothing else in either file.
Update the two tests that pin the old spelling: the darn stencil claim in `wordingClaims` (the `wantAll("the darn writer reads the task from the board entry", ...)` entry) and the first-spawn row of `TestDarnSpec`, whose `wantInPrompt` expects `` `lyx board get fix-the-thing` ``; both now expect the JSON form rendered for the slug.
Grep the repository for the old bare spelling before committing so no other assertion still carries it.

**ImpactSummary:** Two stencil lines and two exact-string test assertions; no Go production code changes.
