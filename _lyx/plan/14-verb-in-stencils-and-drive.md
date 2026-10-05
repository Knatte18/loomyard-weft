# Card 14 — Plan-Write names `decision add`; `ly-drive` relays `parent_notice`

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/claims_test.go`
- `plugins/ly/skills/ly-drive/SKILL.md`
- `internal/loomcli/driveskill_test.go`

**Intent:**
D10: the Plan-Write stencil tells the agent that a scope addition arriving after Discussion is recorded with `lyx loom decision add` before planning on it, and is never written into the plan as an operator addition.
Add this to Step 1, after the sole-input rule, naming `--by` as the source of the addition (`operator` when the operator gave it).
No other agent stencil names the `circling` verbs.
`claims_test.go` gains a claim that the plan stencil names `lyx loom decision add`.

D6: `ly-drive`, at an `awaiting` stop whose envelope carries a non-empty `parent_notice`, with `LYX_PARENT` set and non-empty, sends that notice verbatim to `LYX_PARENT` instead of its generic stop-report line, records the send in the stop report, and hands back as today.
With `LYX_PARENT` unset or empty, or a failed send, the stop report and `reason` are the escalation, as today.
The skill branches on the envelope field only, never on a row, recipe or driver value.
Edit the `awaiting` paragraph and `## Notifying the parent` in `SKILL.md` accordingly; the PR-Gate `awaiting` hand-off, which carries no notice, keeps its current wording.
`driveskill_test.go` asserts the skill names `parent_notice` and still names no loom row.

**ImpactSummary:** Prompt and skill text only, plus two wording assertions.

**Commit:** 14: Plan-Write names decision add; ly-drive relays parent_notice
