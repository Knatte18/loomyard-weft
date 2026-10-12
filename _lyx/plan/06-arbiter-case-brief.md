# Card 6 — arbiter-case-brief

**Edit:**
- `contracts/stencils/loom/loom-template-arbiter.md`
- `contracts/stencils#wordingClaims`
- `internal/loomcli#arbiterPromptInput`
- `internal/loomcli#arbiterPrompt`
- `internal/loomcli#arbiterInput`
- `internal/loomcli#arbiterCall.settleFresh`
- `internal/loomengine#ArbiterAttempt`
- `internal/loomcli#TestArbiterPrompt`
- `internal/loomcli#TestArbiterVerb_SpawnPath`

**Intent:**
D14: the arbiter receives the tackler's case brief as an input file and never as authority.

The arbiter stencil gains the marker `{{.case_brief_path}}`, the literal `none` when absent, named in its leading comment beside `{{.brief_path}}`.
`## Read first` gains a bullet for the tackler's case brief, and the stencil adds lyx's own rule: the brief is a cheaper agent's reading, so check every fact you rely on against its pointer yourself and never take its hypothesis as given.
The arbiter's instructions, allowed actions and verdict format are unchanged.

`arbiterPromptInput` and `arbiterInput` each gain `CaseBriefPath string`.
`arbiterPrompt` fills `case_brief_path` through `pathOrNone`.
`arbiterCall.settleFresh` copies `in.CaseBriefPath` into the prompt input and into the new `ArbiterAttempt.CaseBriefPath` field (JSON `case_brief_path`, omitted when empty), so a restarted driver's re-run names the same brief.
The `lyx loom arbiter` verb's flags are unchanged: the LLM driver passes no brief, and only the Go driver's in-process call (card 13) sets the field.

`wordingClaims`' arbiter row adds `{{.case_brief_path}}` and `never take its hypothesis as given` to its `wantAll`.

Tests:
- `TestArbiterPrompt` gains the brief path to its every-marker row and asserts three `none` fills in the absent row.
- `TestArbiterVerb_SpawnPath` gains a row whose input carries a case brief path, asserting the attempt's `prompt.md` names it and its `attempt.json` records it.

**ImpactSummary:** one stencil marker and a field threaded through the arbiter's prompt, input and attempt record; the CLI verb and every existing attempt record read the same.
