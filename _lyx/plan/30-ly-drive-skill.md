# Card 30 — ly-drive-skill

**Prosa:**
- `plugins/ly/skills/ly-drive/SKILL.md`

**Uses:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Point the driver at the refusal table and at `goto` for its escalations (D3, D11).

- Where the skill describes an escalation or hand-back, add that the stop report names the refusal's way forward, looked up in `contracts/specs/refusal-spec.md` by the refusal text, so the operator reads a next step rather than a diagnosis alone.
- Name `lyx shed goto --to <producer>` as the way forward the driver reports for a `current_producer` no longer in the recipe and for an exhausted bounce budget, and state that the driver never runs `goto` itself: it is an operator and orchestrator verb.

The skill stays recipe-blind: no recipe name, no `.lyx/` path, no recipe-owned command (`cmd/lyx/lydriveskill_test.go` enforces this), so it names `lyx shed goto`, never a recipe subtree's spelling.
Semantic line breaks throughout.
