# Card 15 — lyx creel interrupt claims a covered strand into the pause, then interrupts it

**Create:**
- `plan:internal/creelengine#PauseClaim` -> `type PauseClaim struct`
- `plan:internal/creelcli#hubDelivery.ClaimPause` -> `func (d *hubDelivery) ClaimPause(strand creelengine.Strand) (creelengine.PauseClaim, error)`
- `plan:internal/testkit/creelfake#Delivery.ClaimPause` -> `func (d *Delivery) ClaimPause(strand creelengine.Strand) (creelengine.PauseClaim, error)`
- `plan:internal/testkit/creelfake#Delivery.SetClaimLive` -> `func (d *Delivery) SetClaimLive(name string, live func() (bool, error))`
- `plan:internal/testkit/creelfake#Delivery.Claims` -> `func (d *Delivery) Claims() []string`

**Edit:**
- `internal/creelengine#Delivery`
- `internal/creelengine#Store.interruptTarget`
- `internal/creelengine#Store.Interrupt`
- `internal/creelengine/doc.go`
- `internal/creelcli#interruptCmd`
- `internal/creelcli#injectCmd`
- `internal/testkit/creelfake#Delivery`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelcli#TestHubDelivery`

**Uses:**
- `plan:internal/pauserecord#ClaimSession`
- `internal/pauserecord#Read`
- `internal/testkit/creelfake#Delivery.SetPaused`
- `internal/testkit/creelfake#Delivery.Interrupts`
- `internal/creelcli#newStubbedDelivery`
- `internal/creelcli#pauseRecordPath`

**Intent:**
D7.
`PauseClaim` is the answer of the new `Delivery.ClaimPause`: `Covered` reports that a live entry covered the strand and now owns it, and `StillLive func() (bool, error)` re-reads whether that entry is still live, nil when uncovered.
`Delivery` gains `ClaimPause(strand Strand) (PauseClaim, error)`, used only by the interrupt; `Paused` stays the read-only check the delivery worker holds mail on, and records nothing.
The record write goes through the seam, implemented in `internal/creelcli` (`PATTERN-creel-store`); `creelengine` imports no `pauserecord`.

`interruptTarget` keeps its order with the pause step moved behind the hold: privilege (in `Interrupt`) and orch rules first, then another sender's hold refuses as today, then `ClaimPause`; a claim error refuses as the unreadable record did, `creel: could not read the pause record for <strand>, so nothing was interrupted: <error>`, with nothing recorded.
Under the receiver's pane lock, right before the key, a covered claim calls `StillLive` over the entry the claim returned, which is the entry that owns the strand (card 1): an error refuses the same way, and an entry no longer live sends no key and refuses with `creel: the pause covering <strand> was lifted before the interrupt, so nothing was interrupted; way forward: run "lyx creel interrupt <strand>" again`, which a retry passes as an uncovered interrupt, so the caller is never told a running turn was absent.
Otherwise the guarded interrupt goes as for an uncovered strand.
The claim comes before the key, so an interrupt that dies after it leaves the strand recorded, as tender's pause orders it; a strand that reads no running turn (`ErrNoRunningTurn`) stays recorded, as tender's `no_running_turn` session does.
`Inject` with `--interrupt` reaches the interrupt through `interruptTarget` too, so under a pause it now claims, interrupts and queues the inject, which the worker holds until the resume.
Bound: the interrupt is let through only for a strand a live entry covers at the key, and it records the strand in that entry; no sender joins the privilege rule.
Residual, accepted: `lyx tender resume` never takes creel's pane lock, so a resume landing between the re-read and the key leaves the strand interrupted with no resume line if its line then fails busy.
The sentence of `doc.go`'s Layering section listing what Delivery does gains claiming a covered strand into its pause for the interrupt;
its interrupt and Pause paragraphs, `Store.Interrupt`'s doc comment and the `Long` text of `interruptCmd` and of `injectCmd`'s `--interrupt` say a covered strand is claimed into the pause and interrupted, so `lyx tender resume` gives it its line, and that the unreadable record and hold refusals stay.

`hubDelivery.ClaimPause` resolves the strand's worktree and its `hubgeom.PauseGeometry`, and calls `pauserecord.ClaimSession` with the strand's slug and session: the orch strand (the prime's strand whose parsed role is `agentname.RoleOrch`) as `{GUID, Orch: true}`, the shape the orch watcher writes, so its watcher finds the orch owned and sends no Esc of its own and still posts the resume line; any other strand as `{Strand: name, GUID, Slug}`.
`StillLive` reads the record with `pauserecord.Read` and reports whether a live entry of the claimed entry's scope and pause time remains.
The creelfake `Delivery` gains `ClaimPause`, which answers covered for a strand set paused, records the name of each strand it claims in `Claims` (an uncovered strand is not recorded), returns the paused error as its error, and takes `StillLive` from `SetClaimLive`, defaulting to live.

`TestStore_InjectAndHold`'s row `Interrupt_RefusedUnderPauseOrAnUnreadablePauseRecord` becomes `Interrupt_UnderAPauseClaimsThenInterrupts`: a covered busy strand is claimed once and interrupted, and `Inject` with interrupt under the pause claims, interrupts and queues; a claim whose entry `SetClaimLive` reports gone gets no key and refuses with the lifted-pause message naming `lyx creel interrupt`;
a claim whose `SetClaimLive` answers an error gets no key and refuses with the unreadable-record message; a covered strand whose interrupt answers `ErrNoRunningTurn` keeps its claim; an unreadable record still refuses `Interrupt` and `Inject` with interrupt with nothing claimed or interrupted; and under a pause another sender's hold and the privilege rule still refuse with nothing claimed.
`TestScheduler_Injects` gains a row: a queued inject to a strand set paused is not typed and leaves `Claims()` empty, so the delivery worker under a pause records no strand.
`TestHubDelivery` gains a subtest over a temp record: under a pair entry `ClaimPause` of the pair strand records `{Strand, GUID, Slug}` in it; under a hub entry `ClaimPause` of `ly:orch` records `{GUID, Orch: true}`, and of a non-orch prime strand records `{Strand, GUID, Slug: ""}`; an uncovered strand records nothing and answers uncovered; `StillLive` answers true until the entry is moved to resumed and false after; an undecodable record is an error.

**ImpactSummary:** the creel `Delivery` seam gains one method with its two implementations (creelcli and the creelfake kit); `interruptTarget` stops refusing a covered strand, which also changes `Inject --interrupt` under a pause; the worker's `Paused` path is untouched.
