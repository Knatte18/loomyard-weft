# Fixer report — plan, round 1

## Changes

- **F1** (BLOCKING), card 3: fixed differently in one part.
  `PauseHold` gains `Poll time.Duration` and `Sleep func(ctx context.Context, d time.Duration) error`.
  `Verify` polls `PastGrace` every `Hold.Poll`, and every `TestVerify_PauseHold` row sets a short `Poll` and a fast `Sleep` on the hold `NewPauseHold` returns.
  The fix proposed that `NewPauseHold` leave `Sleep` nil.
  Instead it sets `Sleep` to a timer wait that ends with `ctx`, which is `WaitUncovered`'s own nil default (`internal/pauserecord/hold.go`, `sleepOnTimer`), so production timing is unchanged.
  F7's fix calls `held` from inside the Waiter's `Sleep` wrapper, and that wrapper needs a non-nil wait to delegate to.
- **F2** (BLOCKING), card 3: fixed as proposed.
  A `Wait` that held is followed by a start over from the dirty-tree check, the same shape as the stop path.
  Once `Wait` returns without holding, Verify writes the running marker.
  New row: a Verify entered under a live hub entry, with an untracked file written in while it is held, returns `StatusDirty` and runs no command.
  The `doc.go` paragraph names the start over after a hold at entry.
- **F4** (MEDIUM), card 14: fixed as proposed, with one row kept.
  A second `PausedError` claims again with that error's `Slug`.
  A claim that records the session or finds it owned reports `Moved`; an uncovered second claim gives the ended-pause reason, and a failed one the claim-failed reason.
  No further send is made.
  A new row has the second refusal come from a still-live `p1` entry and asserts the session is listed under `Moved` and recorded in that entry.
  The proposal said to change the existing twice-refused row instead.
  That row is kept, because it is the only test that reaches the uncovered-second-claim branch.
- **F5** (LOW), card 9: fixed as proposed.
  The "SendConditioned is unchanged" sentence is replaced: its replays read coverage, and revive calls it only after reading the slug uncovered under the record lock, so a refusal there is a send error revive reports.
  `typeVerified`'s doc names that path.
- **F6** (LOW), cards 9 and 10: fixed as proposed.
  Card 10 says a refused segment-color line is logged and dropped as any color failure is.
  Card 9's doc.go Pause section and background caller list name that drop.
- **F7** (LOW), card 3: fixed as proposed, by the route the fix itself names: `held` runs only when `WaitUncovered` has slept.
  `Notify` only logs, and the Waiter's `Sleep` calls `held` before its first sleep.
  `WaitUncovered` sleeps only after it finds a covering entry, and its unreadable-record path returns before any sleep.
  The undecodable-record row asserts the marker never reads `paused`.
- **F8** (LOW), card 2: fixed as proposed.
  A failed group SIGTERM is a fail-loud wiring guard in `contracts/specs/refusal-spec.md`'s sense.
  It is reported through `report` with its error and returned from `cmd.Cancel`, and it takes no reaching test.
  The "kills the shell alone" fallback is gone, and "the cancel returns nil" now applies only to a cancel whose SIGTERM was sent.
- **F9** (NIT), card 15: fixed as proposed.
  The Layering-section sentence that lists what Delivery does gains the claim.

## Self-check

- fix-proposal, LOW: card 2's guard path could leave `awaitGone` waiting on a timer that was never started. Fixed as proposed: the guard starts no timer, and `awaitGone` returns at once.
- fix-claims, LOW: card 10's color sentence named only `loadSkillsThenPrompt`, but `Runner.start` also calls `typeStartupColor` (`internal/shuttleengine/run.go:656-658`). Fixed as proposed: the sentence names both callers.
- fix-claims, LOW: card 15 called the Delivery sentence doc.go's "opening sentence", but it sits in the Layering section (`internal/creelengine/doc.go:9`). Fixed as proposed: the card names the Layering section.
- fix-consistency, LOW: card 9's ImpactSummary still said `SendConditioned` gains no refusal. Fixed as proposed: it now says `typeVerified`'s replays, `SendConditioned`'s included, gain the refusal.
- fix-affected-set, LOW: card 9's background caller list left out creel's queued commands, which `internal/creelcli/delivery.go:114-122` routes through `sendVerified`, and card 12 maps the refusal to `ErrBusy`. Fixed as proposed: the list says creel holds the inject and its queued commands.

## Disputed

None.

## Deferred

- **F3** (MEDIUM), card 8 and card 9's PATTERN line.
  The proposed fix is for the parent (`ly:orch`) to record an Added after Discussion entry through `lyx loom decision add`.
  That is a decision-record write, and this round cannot make it:
  - My write surface is `_lyx/plan` plus this report.
  - My instructions forbid sending any session a message about the review, so I cannot ask the parent either.
  - The change cannot be made inside `_lyx/plan`: the overview's Shared Decisions already records both departures, and the finding is that this route does not count.

  The loop owner or `ly:orch` needs to add the decision entry.
  The finding gives its content: the declared `PATTERN-pause-holds-prompts` line is 502 characters, past `pattern.MaxEntryLineChars` (360), and PATTERN.md at the merge base is 15703 bytes, past `pattern.HeadroomBytes` (15564).
  Cards 8 and 9 stay as they are once the record carries it.
