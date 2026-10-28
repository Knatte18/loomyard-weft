# Card 15 — remove-shuttle-send-interrupt

**Delete:**
- `internal/shuttlecli/send.go`
- `internal/shuttlecli/interrupt.go`
- `internal/shuttlecli#TestSendCmd_AppendsMessageTail`
- `internal/shuttlecli#TestInterruptAndSendCmd_FailWithTheGuardsMessage`

**Edit:**
- `internal/shuttlecli#Command`
- `internal/shuttlecli#TestRunCLI_PositionalArgValidation`
- `internal/shuttleengine#ErrNoRunningTurn`
- `internal/shuttleengine#refuseInterruptKey`
- `internal/shuttleengine#Runner.Inject`
- `internal/shuttleengine#Runner.sweepOrphansOpportunistic`
- `internal/shuttleengine/messagetail.go`
- `internal/shuttleengine#TestRun_Interrupt_PlaysEscape`
- `internal/shuttleengine#TestRunner_Inject_RefusesAnyInterruptKey`
- `internal/shuttleengine/claudeengine#Claude.OpenLaunch`
- `internal/creelengine/doc.go`
- `pattern/PATTERN-pane-text-injection.md`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`

**Extends:**
- `cmd/lym#removedVerbs`

**Uses:**
- `cmd/lym#TestRemovedVerbs_NamedNowhere`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
`lym shuttle send` and `lym shuttle interrupt` go (D8); `lym creel inject`, `lym creel interrupt` and `lym creel post --interrupt`, addressed by strand name, cover their uses.
`internal/shuttlecli/send.go` and `internal/shuttlecli/interrupt.go` are deleted, `Command` registers neither verb, and the two tests that drove them are deleted together with any `cli_test.go` helper only they used; `TestRunCLI_PositionalArgValidation` keeps its `state` and `await` rows and drops the send and interrupt ones, and gains one row asserting that `Command()` registers no child named `send` or `interrupt`, so a verb registered again fails; the row reads the command tree and runs no pre-run, so the cwd it starts in cannot mask the answer.
`Runner.Send` and `Runner.Interrupt` stay for shuttle's own callers and creel's delivery, and `WithMessageTail` stays for its smoke-test callers.

Every refusal and doc that names the verbs names the creel verb instead:
`ErrNoRunningTurn`'s way forward becomes `"lym creel post <strand> <text>" gives the agent a turn`; `refuseInterruptKey`'s two refusals name `"lym creel interrupt <strand>"` as the guarded way to send the interrupt key, and `Runner.Inject`'s doc comment follows; `Runner.sweepOrphansOpportunistic`'s and `Claude.OpenLaunch`'s comments, `internal/shuttleengine/messagetail.go`'s file and `MessageTail` comments (the tail a sender writes, appended to no creel post unless the sender writes it), `pattern/PATTERN-pane-text-injection.md`'s `Runner.Inject` bullet, its `ErrPaused` caller list and its enforcement line, `docs/overview.md`'s shuttle row and `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`'s steps name the creel verbs.
`TestRun_Interrupt_PlaysEscape`'s rows expect the `lym creel post` way forward and `TestRunner_Inject_RefusesAnyInterruptKey` expects `lym creel interrupt`, so a reworded message fails on this card.
`cmd/lym#removedVerbs` gains `"lym shuttle send"` and `"lym shuttle interrupt"`, so `TestRemovedVerbs_NamedNowhere` fails on any production source, stencil or skill that names them again.
`internal/creelengine/doc.go`'s `# Unbuilt: the stage 3 remainder` section, which names both verbs, is replaced by the built design: every agent reaches its parent, its seat peers and the orch through creel, with `post --parent`, the expected-reply mark and its chases; the parent-review relay is the Go driver's `parent_notice` post; `lym shuttle send` and `lym shuttle interrupt` are gone, so every operator route into an agent pane runs through creel's journal, queue, pause and hold; its "Later and not planned" paragraph stays.

Bound, stated in the creel doc: the operator loses guid addressing, which a strand name from `lym reed list` replaces; a line injected under a pause is held with its TTL frozen and typed after the resume, where `lym shuttle send` refused it; it is typed only at idle within its TTL; no `MessageTail` is appended unless the sender writes it; and only EXTERNAL, programs and orch strands may inject into another strand.

**ImpactSummary:** Two operator verbs and their two tests go; three refusal messages and the comments and docs that named the verbs are reworded, with the tests asserting them updated on this card.
