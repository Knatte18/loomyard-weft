# Card 8 — The delivery seam tells a hook strand from a pane-mail strand and reads the turn

**Create:**
- `plan:internal/testkit/creelfake#HookStrand` -> `func HookStrand(name string) creelengine.Strand`
- `plan:internal/testkit/creelfake#Delivery.SetTurn` -> `func (d *Delivery) SetTurn(name string, running bool, endedAt time.Time)`
- `plan:internal/creelcli#seedReedStrands` -> `func seedReedStrands(t *testing.T, path string, strands []reedengine.Strand)`

**Edit:**
- `internal/creelengine#Strand`
- `internal/creelengine#Idleness`
- `internal/creelcli#hubDelivery.worktreeStrands`
- `internal/creelcli#hubDelivery.Idle`
- `internal/testkit/creelfake#LiveStrand`
- `internal/testkit/creelfake#Delivery`
- `internal/testkit/creelfake#Delivery.Idle`
- `internal/creelcli#stubOps`
- `internal/creelcli#stubOps.SessionIdle`
- `internal/creelcli#TestHubDelivery`
- `internal/creelengine/doc.go`
- `internal/shuttleengines#ToolBoundaryRenderer`

**Uses:**
- `internal/reedengine#Strand`
- `internal/shuttleengine#IdleProbe`
- `internal/creelcli#newStubbedDelivery`

**Intent:**
The engine, not the strand, decides which path a strand's posts take (added decision "Posts still reach a session without creel hooks"), and D4 needs the seam's no-turn reading (`idle-reading-missing-from-delivery-seam`).

`creelengine.Strand` gains two fields the adapter fills:
`CreelHooks bool`, the strand's reed `CreelHooks` mark (card 1), set when shuttle launched its session with creel's hook delivery, so D4's ring and D5's bell may reach it;
`PaneMail bool`, true when the strand's engine has no creel hook delivery, so its posts keep today's pane path, the notice and the reminder.
`hubDelivery.worktreeStrands` sets `CreelHooks` from the persisted strand, and `PaneMail` true when the persisted `Provider` is empty or `shuttleengines.ToolBoundaryRenderer(provider)` finds no renderer (added decision "A strand with no hook-delivering engine keeps the pane path").
`shuttleengines.ToolBoundaryRenderer`'s doc comment, whose code is unchanged, is rewritten to say it returns the named engine's creel hook-delivery capability, so a false answer means the engine delivers creel mail by no hook; that wording stays true when card 12 changes what the capability renders.
A strand with no provider, one added with `lym reed add`, has an unknown engine, so it is a pane-mail strand and keeps the notice and the reminder; a Codex session, which today exists only as such a hand-added strand, gets its mail that way.
A Claude strand without the mark, a `lym shuttle open` session or one launched before the deploy, is neither: its engine renders hook delivery, so it gets no notice, reminder, ring or bell, and reads its mail with `lym creel read` (D6).
Bound: only a strand whose engine renders creel hook delivery loses the pane path; `shuttleengines` registers only `claude` today, so the unregistered-provider branch is reached only in tests, by providers such as `codex` and `script`, and the empty-provider branch is what hand-added strands reach in production.

`Idleness` gains `NoTurn bool` and `TurnEndedAt time.Time`, documented as `IdleProbe`'s (card 7); `hubDelivery.Idle` copies them from the probe on every answer.

`creelfake.LiveStrand` sets `PaneMail: true`, so every existing scheduler test keeps exercising the pane path it pins; `HookStrand(name)` returns a live strand with `CreelHooks: true` and `PaneMail: false`.
`creelfake.Delivery` gains a per-strand turn script: `SetTurn(name, running, endedAt)` makes `Idle` answer `NoTurn: !running` and `TurnEndedAt: endedAt` for that strand, on its scripted-stall answer as on its idle one, so a draft stall beside a scripted no-turn reads not idle with `NoTurn` true; an unscripted strand answers `NoTurn` equal to its idle answer and a zero time.
The package doc of creelfake names the turn script beside the idle flags.

`stubOps` gains `noTurn bool` and `turnEndedAt time.Time`, which `stubOps.SessionIdle` returns on its probe.
`seedReedStrands` persists whole reed strands, provider and mark included, as `seedReedState` persists named ones.
`TestHubDelivery` gains rows:
`Strands` reports a strand seeded with provider `claude` and the mark as `CreelHooks` true and `PaneMail` false, one with provider `codex`, which `shuttleengines` does not register, as `PaneMail` true, one with no provider as `PaneMail` true and `CreelHooks` false, and one with provider `claude` and no mark as neither;
`Idle` copies a stub's `noTurn` and `turnEndedAt` onto its answer for an idle and a draft-held probe.
The doc's layering paragraph names the two strand fields and the no-turn reading the seam now answers.

**ImpactSummary:** two seam structs gain fields the adapter and the fake fill; the scheduler reads them only from card 9 on, and no existing caller changes behaviour.
