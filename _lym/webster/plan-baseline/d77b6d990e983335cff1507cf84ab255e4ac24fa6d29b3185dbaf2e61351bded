# Card 17 — Docs and stencils describe the hook delivery

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `internal/orchcli/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`
- `internal/creelengine/doc.go`
- `internal/orchengine/doc.go`
- `internal/orchengine#Watcher.queueResumeLine`
- `internal/creelcli#forwardCmd`
- `internal/creelcli#holdCmd`
- `internal/creelengine#Store.Hold`

**Intent:**
D11: an orch told to wait for a reminder line would wait for nothing, and `CLAUDE.md` has the docs follow the code.
Each card before this one updated the doc of the package it changed; this card rewrites the texts that describe the whole flow across packages.

The orch stencil's step 4 says the wake-up is creel mail delivered by the session's hooks: a `[creel] #<id> from "<sender>": <text>` line arrives in full under a tool result or as the session's turn ends, is already read, and is answered with `lym creel post --reply <id> "<text>"` when it asks for an answer; an idle orch is woken first by a fixed `[creel] Mail is waiting for you.` line and is told to end that turn, after which the mail shows; mail from `EXTERNAL` is answered in the reply text itself; `lym creel read` is needed only for mail the hooks could not hand over, which `lym creel inbox` shows.
The sentence on `[batten notice]` text stays.
`internal/orchcli/doc.go`'s second rule says a batten notice arrives in full as its own quoted `[creel]` line through the orch session's hooks, whichever claim it was posted under, and drops the reminder.
`docs/overview.md`'s creel row and module-table lines describe the hook delivery: the PostToolUse and Stop blocks, the idle waiter and bell, the subagent filter, `post --interrupt`, the pane path kept only for a strand whose engine is unknown or has no hook delivery, and the trust sentence on every Claude session shuttle launches.
`tools/sandbox/SANDBOX-CREEL-SUITE.md` keeps its scenarios: every strand it adds comes from `lym reed add`, has no provider and so keeps the pane path (added decision "A strand with no hook-delivering engine keeps the pane path"), so its typed notices and reminders still hold; its intro gains a sentence saying a Claude session shuttle launches gets its mail through its hooks instead, which no scenario of the suite drives.
The creelengine package doc's Orch strands section says an orch strand gets its mail through its hooks like any marked strand, and its "Unbuilt" stage 3 section keeps naming `creel-agents` unchanged.
The orchengine package doc's passages on mail kept unreminded during a cycle, old mail reminded after a failed withdraw, and the resume line reminded once the pause lifts, and `queueResumeLine`'s doc comment, are rewritten to say the orch's mail arrives through its hooks, or the bell when it is idle, once no pause or hold mutes it.
`forwardCmd`'s Long, which says the receiver gets a reminder, and `holdCmd`'s Long and `Store.Hold`'s doc comment, which say only the owner's mail is reminded, say instead that the receiver gets the mail through its hooks, or by its notice or reminder when it is a pane-mail strand, and that under a hold only the owner's mail is delivered.

**ImpactSummary:** prose only, in one stencil, three package docs, two command help texts, two doc comments and two markdown docs; no build or test reads any changed sentence.
