# Card 5 — The plan route and push rule name the foreground gate

**Edit:**
- `internal/planparser#checkVerifyModuleWide`
- `internal/planparser/validate_test.go`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/claims_test.go`
- `CLAUDE.md`

**Intent:**
Decision record D7: every text that names `lyx gate test` "run as a background Bash call" re-creates the wake pattern for whoever reads it, so each names the foreground route under the gate rule instead.

`checkVerifyModuleWide`: the finding's `Detail` ends "use `lyx gate test [-C <module>] [--tags <tags>] <packages>` over the card's own packages, run in the foreground under the gate rule in your system prompt", and the function's doc comment's "run as a background Bash call" becomes "run in the foreground under the gate rule in the agent's system prompt".
The way forward is a command the agents' settings allow (PATTERN-no-denied-recovery).
The refusal is reached by any card whose `**Verify:**` holds a module-wide or tmux- or llm-tier `go` command, as the existing rows of `TestValidate_VerifyModuleWide` show.

`TestValidate_VerifyModuleWide` gains an assertion: every `verify-module-wide` finding's `Detail` contains "run in the foreground under the gate rule in your system prompt" and not "background".

`loom-plan-spec.md`: row 38 (`verify-module-wide`)'s way forward, in the same commit as the refusal text (PATTERN-refusal-way-forward), becomes "Its way forward is `lyx gate test [-C <module>] [--tags <tags>] <packages>` over the card's own packages, run in the foreground under the gate rule in the agent's system prompt."

`loom-template-plan.md`: the card `**Verify:**` paragraph's "`lyx gate test [-C <module>] [--tags <tags>] <packages>`, run as a background Bash call, is the route" becomes "`lyx gate test [-C <module>] [--tags <tags>] <packages>`, run in the foreground under the gate rule in your system prompt, is the route".
`claims_test.go`'s plan-template row that pins the old phrase pins the new one.

`CLAUDE.md`'s push rule: "each run as its own background Bash call" becomes "each run as its own Bash call as the gate rule in the session's system prompt says".

After this card `git grep "background Bash call"` outside `_lyx/` finds no text about `lyx gate test`.

**ImpactSummary:** one plan-gate refusal's wording, its spec row, the plan stencil sentence, one claims row and the CLAUDE.md push rule change; no check logic changes.

**Commit:** 5: the plan route and push rule name the foreground gate
