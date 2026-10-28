# Card 9 — The pane path types posts only into a strand whose engine has no hook delivery

**Create:**
- `plan:internal/creelcli#smokeHub.addProviderStrand` -> `func (s *smokeHub) addProviderStrand(t *testing.T, role, command, provider string) (name, guid string)`

**Edit:**
- `internal/creelengine#worker.probe`
- `internal/creelengine#Scheduler`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelcli#TestSmokeCreelFakeProviderPane`

**Uses:**
- `plan:internal/testkit/creelfake#HookStrand`
- `internal/testkit/creelfake#LiveStrand`
- `internal/creelengine_test#awaitTyped`
- `internal/creelcli#smokeHub`
- `internal/reedengine#AddSpec`

**Intent:**
D6 as narrowed by the added decision "Posts still reach a session without creel hooks": the pane path for posts, the notice line, the reminder with its backoff and notice cap, and the pane reservation before typing, stays only for a strand whose engine has no creel hook delivery, `Strand.PaneMail` (card 8).
Posts to a strand whose engine renders creel hook delivery, a Claude strand shuttle launched, are delivered by its hooks; no message body is typed into its pane.

`worker.probe` keeps typing queued injects, command injects included, into every live strand first, as today.
With no inject due it types the oldest notice or the reminder only when `strand.PaneMail` is set; for any other strand it types nothing and reports nothing delivered, so the worker backs off as after any probe that delivers nothing and retires once the strand's mail is read.
A strand marked `CreelHooks` gets its idle wake from card 10 and its bell from card 11; a strand added by hand has no provider and is a pane-mail strand (card 8), and a Claude strand without the mark has neither field and keeps its mail for `lym creel read`.
`Scheduler`'s doc comment says what it types: queued injects into every strand, and notices and reminders into a pane-mail strand.
The reservation, `reserveNotice`, `releaseNotice` and `markNoticeRead` are unchanged and act only on the pane-mail path that remains.
Old journals' `reminded` and reservation records still fold and act as before (`PATTERN-tolerant-records`).

`TestScheduler_Reminders` gains a row: a post from a program and one from a strand to a `HookStrand` and to a live strand whose `PaneMail` is false type nothing into either pane over several probe waits, while an inject queued after them to each is typed; every existing row runs on `LiveStrand`, whose `PaneMail` is true, and keeps pinning the pane path.
`TestSmokeCreelFakeProviderPane` keeps pinning the pane path end to end: it adds its strand through `addProviderStrand` with the provider `script`, which `shuttleengines` does not register, so the strand is a pane-mail strand and the carried line is still typed.
`addProviderStrand` adds the strand through the smoke hub's reed engine with `AddSpec.Provider` set, since `lym reed add` names no provider.
`TestSmokeCreelClaudeStrand` is unchanged: its claude strand is added by hand, has no provider and so stays a pane-mail strand, whose posted message is still typed into its pane.
The doc's delivery-loop and mail sections say the pane path types notices and reminders only into a pane-mail strand.

**ImpactSummary:** the worker's choice of what to type narrows to injects for every strand but a pane-mail one; the scheduler's other callers are unchanged and every test of the old pane path runs on pane-mail strands.
