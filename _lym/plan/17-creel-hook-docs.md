# Card 17 — Docs and stencils describe the hook delivery

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `internal/orchcli/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`
- `internal/creelengine/doc.go`

**Intent:**
D11: an orch told to wait for a reminder line would wait for nothing, and `CLAUDE.md` has the docs follow the code.
Each card before this one updated the doc of the package it changed; this card rewrites the texts that describe the whole flow across packages.

The orch stencil's step 4 says the wake-up is creel mail delivered by the session's hooks: a `[creel] #<id> from "<sender>": <text>` line arrives in full under a tool result or as the session's turn ends, is already read, and is answered with `lym creel post --reply <id> "<text>"` when it asks for an answer; an idle orch is woken first by a fixed `[creel] Mail is waiting for you.` line and is told to end that turn, after which the mail shows; mail from `EXTERNAL` is answered in the reply text itself; `lym creel read` is needed only for mail the hooks could not hand over, which `lym creel inbox` shows.
The sentence on `[batten notice]` text stays.
`internal/orchcli/doc.go`'s second rule says a batten notice arrives in full as its own quoted `[creel]` line through the orch session's hooks, whichever claim it was posted under, and drops the reminder.
`docs/overview.md`'s creel row and module-table lines describe the hook delivery: the PostToolUse and Stop blocks, the idle waiter and bell, the subagent filter, `post --interrupt`, the pane path kept only for a strand whose engine has no hook delivery, and the trust sentence on every Claude session shuttle launches.
`tools/sandbox/SANDBOX-CREEL-SUITE.md` follows throughout, not in one scenario: its intro and judging notes stop expecting reminders typed into a Claude strand's pane, S9 is rewritten to the new behaviour of a hand-added strand (no message line is typed into its pane, its mail stays for `lym creel read`, an inject is still typed, and a typed message line is the failure), and the busy-turn and reminder-backoff scenarios, which a Claude strand no longer reaches, are rewritten to the hook delivery (the mail shows as a block under a tool result or at the turn's end) or dropped where no hook behaviour replaces them.
The creelengine package doc's Orch strands section says an orch strand gets its mail through its hooks like any marked strand, and its "Unbuilt" stage 3 section keeps naming `creel-agents` unchanged.

**ImpactSummary:** prose only, in one stencil, two package docs and two markdown docs; no build or test reads any changed sentence.
