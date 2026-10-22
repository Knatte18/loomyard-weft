---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/03-verifytree-pause-hold.md
    summary: TestVerify_PauseHold waits out production durations, since PauseHold gives no seam for Verify's one-second PastGrace poll or for WaitUncovered's real one-to-five-second sleeps (PATTERN-test-speed)
    fix: In card 3's Intent, give PauseHold two more fields, Poll time.Duration (the interval Verify polls PastGrace at while the command runs) and Sleep func(ctx context.Context, d time.Duration) error (passed as the Waiter's Sleep). NewPauseHold sets Poll to one second and leaves Sleep nil, so production keeps today's values. TestVerify_PauseHold sets a short Poll and a fast Sleep on the hold NewPauseHold returns, so no row waits out a production interval. Add no targets.
    origin: handler
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/03-verifytree-pause-hold.md
    summary: A Verify entered under a covering pause runs its dirty and skip checks before Hold.Wait and does not rerun them after the resume, so a tree dirtied during the pause is verified (PATTERN-verified-tree, no site verifies a dirty tree)
    fix: In card 3's Intent, after a Hold.Wait that waited (held was called), Verify starts over from its dirty-tree check, exactly as the stop path does, before it writes the running marker. Add a TestVerify_PauseHold row in which an untracked file is written into the worktree while a Verify entered under a live hub entry is held; after the resume it returns StatusDirty and runs no command.
    origin: lens:guard-reach
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/08-verified-tree-hold-pattern.md
    summary: Card 8 cuts clauses from eight undeclared PATTERN entries and card 9 writes a PATTERN-pause-holds-prompts overview line other than the declared one; both are design calls made after the Discussion that the decision record does not carry
    fix: Ask the parent (ly:orch) to record, through lyx loom decision add, an Added after Discussion entry. It states that the declared PATTERN-pause-holds-prompts line is 502 characters, past pattern.MaxEntryLineChars (360), so the overview carries card 9's 262-byte summary and the background carries the declared text. It also states that PATTERN.md at the merge base (15703 bytes) is past pattern.HeadroomBytes (15564), so card 8 moves the named trailing clauses of the eight listed entries word for word into their background files. Keep cards 8 and 9 as they are once the record carries the entry.
    origin: handler
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/plan/14-tender-resume-claims-paused.md
    summary: A resume line refused a second time by a live entry is reported NoLine with nothing claimed, so the session stays under that entry unowned and its later resume never gives it the line, against D2's rule that such a send claims the session
    fix: In card 14's Intent, when the resend after an uncovered claim is refused again with a PausedError, call pauserecord.ClaimSession once more with that error's slug. A claim that records or finds the session owned reports it under Moved, and only an uncovered or failed second claim reports NoLine, with no further send. Change the twice-refused TestResume row so its second refusal comes from an entry that is still live, and assert the session is listed under Moved and recorded in that entry.
    origin: lens:way-forward
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/09-shuttle-send-refuses-covered.md
    summary: Card 9 says SendConditioned is unchanged, but its context comes from Runner.newSendContext, which now sets paused, and it calls typeVerified directly, so tender revive's conditioned send can be refused with ErrPaused at a replay
    fix: In card 9's Intent, replace the sentence that SendConditioned is unchanged with one saying its replays also read coverage. Say that tender revive calls it only after reading the slug uncovered under the record lock, so a refusal there is a send error revive reports as it reports any other. Name that path in typeVerified's doc. Add no targets.
    origin: lens:affected-set
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/10-shuttle-start-holds-paused-prompt.md
    summary: A pause landing after the start's wait refuses typeStartupColor's line through typeColor, sendSequence and sendVerified, and the color is dropped with a Warn while card 10 holds only the prompt and skill-load lines
    fix: In card 10's Intent, state that a segment-color line refused with ErrPaused is logged and dropped as any other color failure is, since it is cosmetic and starts no turn. Have card 9's doc.go and background list of ErrPaused callers name that drop. Add no targets.
    origin: lens:affected-set
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/03-verifytree-pause-hold.md
    summary: The Wait that NewPauseHold builds calls held from the Waiter's Notify, which WaitUncovered also calls for an unreadable record before it returns nil, so an undecodable record writes a paused marker though nothing holds
    fix: In card 3's Intent, have Notify call held only when the line is the held line, not the unreadable-record line, for example by having Wait read whether WaitUncovered slept before calling held. Have the undecodable-record row of TestVerify_PauseHold assert the marker never reads paused.
    origin: lens:guard-reach
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/02-verify-cancel-terminates-group.md
    summary: The prescribed branch where a failed group signal kills the shell alone has no reaching input, because cmd.Cancel runs only while the unreaped shell keeps its group alive, and the card names no test for it
    fix: In card 2's Intent, write the failed-group-signal branch as a fail-loud wiring guard in refusal-spec.md's sense, reported through report with its error, or drop it. State that it takes no reaching test.
    origin: lens:guard-reach
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/plan/15-creel-interrupt-claims-covered.md
    summary: creelengine/doc.go's opening seam sentence lists the Delivery methods and goes stale with ClaimPause, while card 15 rewrites only the interrupt and Pause paragraphs
    fix: In card 15's Intent, add that doc.go's opening sentence listing what Delivery does also names claiming a covered strand into its pause for the interrupt.
    origin: lens:affected-set
---

### [BLOCKING] TestVerify_PauseHold waits out production durations

**Location:** `_lyx/plan/03-verifytree-pause-hold.md`

**Issue:** Card 3 has `Verify` poll `Hold.PastGrace` "once a second", with no interval field.
`NewPauseHold` builds its `Wait` over `pauserecord.WaitUncovered` with a `Waiter` that sets only `Notify` and `SlugEntryExempt`.
`WaitUncovered`'s nil `Sleep` waits on real timers from one second, doubling to five (`internal/pauserecord/hold.go`, `holdPollFloor`, `holdPollCeiling`, `sleepOnTimer`).
`TestVerify_PauseHold`'s rows build their hold with `NewPauseHold`, and each one waits at least one real poll:

- a held entry that is then resumed;
- a stop past the grace end, a hold, and a rerun;
- the dirty and skipped reruns;
- the exempt row.

`PATTERN-test-speed`'s "Time is injectable" section requires that a production interval a test would wait out be a struct field or a parameter, and says no test in any tier waits out a production duration.

### [BLOCKING] A Verify entered under a pause skips the dirty check after the resume

**Location:** `_lyx/plan/03-verifytree-pause-hold.md`

**Issue:** Card 3 calls `Hold.Wait` "after the dirty and skip checks and before writing its running marker", then "writes the running marker as today and runs the command".
In `internal/verifytree/verify.go#Verify`, the dirty check, the HEAD tree and commit reads, and the skip check all come before the marker write.
So a pause of any length sits between those checks and the command.
A tree dirtied during that pause is verified anyway, against `pattern/PATTERN-verified-tree.md`'s rule that no site verifies a dirty tree.
A record that names HEAD's tree, written during the wait, is not read either, so the pass is not skipped.
The card's own stop path already restarts from the dirty check, which is the shape the fix gives the entry path too.
D4's "after the dirty and skip checks" places the wait and is not contradicted by rechecking after it.

### [MEDIUM] Card 8's undeclared PATTERN cuts and card 9's departing line are unrecorded design calls

**Location:** `_lyx/plan/08-verified-tree-hold-pattern.md`

**Issue:** The decision record declares two PATTERN changes and nothing else.
Card 8 edits eight other entries' overview lines and backgrounds.
Card 9 writes a `PATTERN-pause-holds-prompts` overview line that differs from the declared text.
I checked both constraints the plan cites:

- `PATTERN.md` is 15703 bytes, over `HeadroomBytes` (`MaxOverviewBytes * 95 / 100`).
- `pattern.Headroom` refuses any growth past it (`internal/pattern/headroom.go:12`).
- The declared entry line measures 502 characters against `MaxEntryLineChars` 360.

So some departure is forced.
The eight clauses total 313 bytes, the growth totals 292, and each clause moves word for word into a background file that exists.
No PATTERN rule is broken, and no test reads the cut text.
But which entries lose clauses, and what the new overview line says, are design calls made after the Discussion.
The plan writer's stencil (`loom-template-plan.md`, scope rules) routes such a call through the parent and plans on it only once the record carries it.
The overview's `## Shared Decisions` records the call without that route.

### [MEDIUM] A twice-refused resume line strands the session

**Location:** `_lyx/plan/14-tender-resume-claims-paused.md`

**Issue:** On a `PausedError`, card 14 claims the session.
When that claim answers uncovered, it sends once more.
A second `PausedError` there means a newer live entry now covers the session, yet the card reports `NoLine` with the "held by a pause that has since ended and a second one" reason and claims nothing.
`liftEntries` gives lines only to an entry's own `Sessions` (`internal/tenderengine/resume.go`), so that entry's later resume never gives this session its line.
D2 says "a send refused because another live entry covers the session claims the session into that entry through D6".

### [LOW] SendConditioned's replays now read coverage, against card 9's claim

**Location:** `_lyx/plan/09-shuttle-send-refuses-covered.md`

**Issue:** `SendConditioned` (`internal/shuttleengine/conditionedsend.go:43-50`) builds its context with `Runner.newSendContext`, which card 9 has set `paused`, and it calls `typeVerified` directly.
`typeVerified`'s new re-read before each replay therefore reaches tender revive's conditioned send.
Revive runs the call inside its record `Update` (`internal/tenderengine/resume.go`, `reviveSession`), and `pauserecord.Read` takes no lock, so nothing deadlocks.
The card's statement that the path is unchanged is still wrong.

### [LOW] The start's segment-color line is refused and dropped under a late pause

**Location:** `_lyx/plan/10-shuttle-start-holds-paused-prompt.md`

**Issue:** `typeStartupColor` (`internal/shuttleengine/run.go:669-676`) sends through `typeColor`, then `sendSequence`, then `sendVerified` on the paste engine (`send.go:547-556`).
A pause that lands after `Runner.start`'s wait therefore refuses the line with `ErrPaused`.
The refusal is logged at Warn and the color is dropped.
Card 10 holds only the prompt and skill-load lines, and card 9's list of `ErrPaused` callers leaves this one out.

### [LOW] An unreadable record writes a paused marker

**Location:** `_lyx/plan/03-verifytree-pause-hold.md`

**Issue:** `WaitUncovered` calls `notify` with "pause record unreadable, not holding" and returns nil when `Read` fails (`internal/pauserecord/hold.go`).
Card 3's `Notify` both logs the line and calls `held`, so that call writes a `paused` marker for a Verify nothing holds.
The running marker overwrites it at once, which makes the effect brief, but the card's contract that `held` runs "when it starts waiting" is not what the wiring does.

### [LOW] Card 2's failed-group-signal branch has no reaching input

**Location:** `_lyx/plan/02-verify-cancel-terminates-group.md`

**Issue:** `cmd.Cancel` runs only before `Wait` has reaped the shell.
The unreaped leader keeps its group in existence, so `syscall.Kill(-pid, SIGTERM)` has no realistic failure there.
Card 2 prescribes a fallback branch for that failure and names no test that reaches it.
The rubric's reachability bullet admits such a branch only as a fail-loud wiring guard.

### [NIT] creelengine's seam sentence goes stale

**Location:** `_lyx/plan/15-creel-interrupt-claims-covered.md`

**Issue:** `internal/creelengine/doc.go:9` lists what `Delivery` does.
Card 15 adds `ClaimPause` but rewrites only the doc's interrupt and Pause paragraphs, so that sentence no longer names every method.

## Rejected

- way-forward F2, which says card 15's `StillLive` error message misleads: the text "so nothing was interrupted" is true in that state, and the strand staying recorded is the claim-before-key order D7 prescribes. Creel is also outside `PATTERN-refusal-way-forward`'s bound modules.
- Handler, card 1 returning the owning entry rather than the covering one when another live entry already owns the session: D7 re-reads "the claimed entry", and the owning entry is the one whose resume gives the line, so this reads D6 consistently.
- Handler, card 15 changing `lyx creel inject --interrupt` under a pause: it is the same `interruptTarget` step D7 changes, and the overview's Shared Decisions records that.
- Focus 2: `planVerifySites`' four files all gain a `Hold:` literal or a `.Hold =` assignment from cards 5, 6 and 7, all of which precede card 8, so the tripwire is satisfied. Not a defect.
- Focus 6: every re-signed member carries the re-sign arrow (cards 5, 9, 10, 11, 13 and 14), and no new test repeats an existing table. Not a defect.
