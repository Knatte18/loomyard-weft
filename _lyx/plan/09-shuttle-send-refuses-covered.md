# Card 9 — shuttle's typing chokepoint refuses a session a pause covers

**Create:**
- `plan:internal/shuttleengine#ErrPaused` -> `var ErrPaused = errors.New("shuttle: a pause covers the session")`
- `plan:internal/shuttleengine#PausedError` -> `type PausedError struct`
- `plan:internal/shuttleengine#PausedError.Error` -> `func (e *PausedError) Error() string`
- `plan:internal/shuttleengine#PausedError.Unwrap` -> `func (e *PausedError) Unwrap() error`
- `plan:internal/shuttleengine#Runner.targetSlug` -> `func (r *Runner) targetSlug(guid, name string) string`
- `plan:internal/shuttleengine#Runner.pausedTarget` -> `func (r *Runner) pausedTarget(guid, name string) error`
- `plan:internal/shuttleengine#coverScope` -> `func coverScope(t *testing.T, scope string, pausedAt time.Time) (string, func(resumedAt time.Time))`
- `plan:internal/shuttleengine#TestSend_RefusesACoveredTarget` -> `func TestSend_RefusesACoveredTarget(t *testing.T)`
- `pattern/PATTERN-pause-holds-prompts.md`

**Edit:**
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#Runner.SetPause`
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#Run.newSendContext`
- `internal/shuttleengine#Runner.newSendContext`
- `internal/shuttleengine#Runner.recordlessSendContext` -> `func (r *Runner) recordlessSendContext(guid, name, provider string) (sendContext, error)`
- `internal/shuttleengine#Runner.Send`
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#typeVerified`
- `internal/shuttleengine/doc.go`
- `PATTERN.md`

**Uses:**
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#echoReed`
- `internal/shuttleengine#sessionFakeEngine`
- `internal/shuttleengine#liftClock`
- `internal/shuttleengine#newFakeClock`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/pauserecord#ReadCoverage`
- `internal/pauserecord#Update`

**Intent:**
D1.
`ErrPaused` marks a send refused because a live pause entry covers the target session; nothing was typed.
`PausedError` is that refusal: `Slug`, the slug the coverage read judged; `Entry`, the covering entry; `Err`, the full error, `fmt.Errorf("%w: strand %q: %w", ErrPaused, guid, pauserecord.CoveredError(entry, slug, "retry the send"))`, so the text names the resume that lifts the hold; `Error` returns `Err`'s message and `Unwrap` returns `Err`, so `errors.Is(err, ErrPaused)` holds and a caller that renders its own retry reads `Entry` and `Slug`.
`ErrPaused` is its own sentinel, never wrapping `ErrSessionBusy`, so a caller that holds a busy send does not mistake a paused one for it.

`Runner.targetSlug` returns the slug of the target strand's own name as reed reports it, parsed with `agentname.Parse`, a prime name giving the empty slug, which only a hub entry covers.
`name` is the name the caller already holds from reed: the run record's `StrandName`, which reed formed at `AddStrand`, or the live strand's name `liveStrand` read for a recordless send; only when it is empty does `targetSlug` call `reed.Status` and find the strand by guid, so a send or tick with a known name takes no extra reed call.
A name that does not parse, and a strand reed does not list, fall back to the runner's geometry slug; a failed `Status` falls back the same way and logs one Warn per change of error.
`Runner.pausedTarget` returns nil for a runner told no geometry, which is what tests and the orch runner use; otherwise it reads `pauserecord.ReadCoverage(r.pause.RecordPath, slug, r.clock.Now())` for the target's slug and returns a `*PausedError` when covered.
An unreadable record reads uncovered and logs one Warn per change of error, as `observePause` does, so a corrupt file never freezes sends.
`Runner` gains the fields that keep the last warned text of each of the two readings under a mutex, so a persistent error warns once and a changed one warns again.

`sendContext` gains a `paused func() error` field, which every constructor (`Run.newSendContext`, `Runner.newSendContext`, `Runner.recordlessSendContext`) sets to the runner's `pausedTarget` over the context's guid and the name it holds (the state's `StrandName`, or for `recordlessSendContext` the new `name` parameter, which its one caller `Runner.Send` passes from the `liveStrand` reading it already takes).
`sendVerified` calls it after `awaitIdleSession` returns and immediately before the retry check and `typeVerified`, and returns its refusal with nothing typed, since a tender pause's grace is the window in which a send waits for idle.
`typeVerified` calls it again before each replay of its choreography (`try > 0`) and returns the refusal with that replay untyped.
A nil `paused` reads uncovered.
Every shuttle line but `Runner.SendConditioned`'s passes `sendVerified`, `sendSequence`'s paste path included.
`SendConditioned` builds its context with `Runner.newSendContext` and calls `typeVerified` directly, so its replays also read coverage; its only caller, tender revive, calls it only after reading the slug uncovered under the record lock, so a refusal there is a send error revive reports as it reports any other.
`typeVerified`'s doc names that path.
Bound: the check only refuses; it types nothing, records nothing and never waits.
Keys stay outside it: `Interrupt`, `InterruptRestoring`, `Inject` and the pause's own Esc are key deliveries under `PATTERN-pane-text-injection`.
`SetPause`'s doc says the geometry is also what every send judges its target's coverage by.
`doc.go`'s Pause section gains the send refusal, its target slug, its two fallbacks, the runners told no geometry, and the start's segment-color line, which a refusal drops with a Warn as any color failure (card 10).
Until cards 10 to 14, a caller that gets `ErrPaused` treats it as any other send failure.

`coverScope` writes a live entry of `scope` paused at `pausedAt` into a fresh temp record and returns the record path and the resume function, beside `coverHub`, for a row that needs a slug entry.
`TestSend_RefusesACoveredTarget` is table-driven through `Runner.Send` and `Run.Send` over `newFixture`, the strand named on its run record's `StrandName` or, for the recordless row, in the reed `Status` the fixture scripts, asserting the error is a `*PausedError` with the named slug and that nothing was typed or pasted:
a hub entry refuses a send into a strand named `ly:p1:impl` from a runner whose geometry slug is `other`;
a slug entry of `p1` refuses that same send, the runner's own slug reading uncovered;
a slug entry of `p1` does not refuse a send into a prime strand named `ly:impl`, and a hub entry does;
a strand whose name does not parse falls back to the runner's slug;
a slug entry of `p1` refuses a `Runner.Send` into a live strand named `ly:p1:impl` that no run record names, through `recordlessSendContext`;
a pause added while the send waits for an idle session (the pane busy until a `liftClock` sleep adds the entry and frees it) refuses with nothing typed;
a pause added after the first typing of an engine without the paste capability, whose first typing the pane swallows (a `liftClock` sleep inside the verify polls adds the entry), refuses the replay with one typing played;
a runner told no geometry types under a live hub entry, the runner shape `lyx orch start` uses, whose wiring card 12 pins;
an undecodable record types and logs one Warn across two sends.
The test also calls `Runner.targetSlug` with an empty name over a `fakeReed` whose `StatusErr` is set, twice: both answer the runner's geometry slug, and one Warn is logged.

The new `PATTERN.md` entry goes in the Agents and prompts section after `PATTERN-pane-text-injection`, verbatim:

```markdown
- `PATTERN-pause-holds-prompts` — Typing a line into a covered session: shuttle's send refuses by the target's slug; its caller holds the line or claims the session; only the orch runner reads uncovered. — [background](pattern/PATTERN-pause-holds-prompts.md)
```

Its background file `pattern/PATTERN-pause-holds-prompts.md` carries the declared entry text in full, as the rule, and lists the callers and what each does with `ErrPaused`: the gate loop holds, the run start and the deferred prompt wait and resend, the seat notice sender retries, creel holds the inject and its queued commands, tender resume claims the session, tender revive's conditioned send reports the refusal as a send error, the start's segment-color line is logged and dropped, and `lyx loom resume` and `lyx shuttle send` refuse with the pause's way forward; and that the tender and creel runners are told the hub's geometry with an empty slug.
The overview line is shorter than the declared text because of `pattern-entry-line-cap` and the headroom; Shared Decisions records it.

**ImpactSummary:** the one typing chokepoint (`sendVerified`, reached by every shuttle send but `SendConditioned`) gains a coverage refusal, and `typeVerified`'s replays, `SendConditioned`'s included, gain it too; only runners told a geometry can refuse, so tests and the orch runner are unchanged; one new PATTERN entry with its background.
