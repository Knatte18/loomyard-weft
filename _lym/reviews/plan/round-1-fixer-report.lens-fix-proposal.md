```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the review proposals the fixer carried out (F4, F6, F7, F8, F9, F10, F11, F12, F13, and the proposed parts of F1, F2 and F5).

**[MEDIUM] F6 as proposed still drops other runs' owed transitions from the parent's records branch**
- Changed text: card 3, steps 2 and 5. Step 2 archives the parent pair through `ArchiveOwnerTips`; step 5 resets only the rejected branch with `reset --keep` to origin's new tip.
- The case: this machine's parent records branch holds an owed transition of run X, which origin lacks and which was never pushed. Step 4 commits the takeover entry on top and pushes. Origin has moved, so the push is rejected. Step 5 resets that branch to origin's tip, so X's transition leaves the live branch and survives only under an `archive-owner/` tag.
- Card 9's step 4 catch-up then finds nothing owed for X and pushes nothing. X's status on origin and locally falls back to the older state.
- Evidence: D6's criterion that the catch-up "pushes the owner's owed transitions without resetting them". The reviewer's summary names exactly this data loss, and archiving only makes it recoverable by hand. A sound change: step 5 rebases this machine's local commits that origin lacks onto the new tip, or rewrites the entry on top of them. A blind reset is the wrong tool here.

**[MEDIUM] F2: a take-hold killed before the fix task's takeover strands the fix task on the re-run**
- Changed text: card 9, step 2. `FixTask` is exempt when `WriteAdopt` kept this machine's hold (`Kept`); under a take-hold rewrite it is judged like any other run.
- The case:
  1. `lym adopt --take-hold alpha` rewrites the `adopt-fix` hold to this machine in step 1.
  2. The adopt is killed before `TakeOver` of the fix task's run.
  3. The plain re-run `lym adopt alpha` reads this machine's same-set hold, so `Kept` is true and the fix task is exempt.
  4. Its run is never moved, and it carries no takeover entry naming this machine. That is the strand F2 set out to fix.
- Evidence: card 9's own sentence "an adopt killed after step 1 still moves the runs it took", and D6 step 2's kill guarantee.
- The reviewer's `TookOver` field has the same gap, because the re-run is not a take-hold write either. A sound change has to persist the fact in tracked state. For example, the hold records the machine that filed `FixTask`, and the exemption applies only when that machine reads as this one.

**[LOW] F9's header wording pins an enumerated writer list**
- Changed text: card 2, "holdwrite.go's file header names `WriteHold`, `WriteAdopt`, `FixHold` and `ClearHold` as the writers of `holds`."
- That is an enumerated-consumer list in a file header, which goes stale with the next writer. The doc.go sentence beside it does the same.
- Evidence: the scribe `code-quality` rule "No enumerated-consumer lists". A sound change: the header says holdwrite.go holds boardengine's writers of `holds` and drops the "only writer" claim.

**[LOW] F7's fronttask hand step is a lookup, not a step**
- Changed text: card 8, "`actions` row `adopt-check` words its hand step as `lym beam status`".
- `fronttask.go:318` renders the failure as "way forward: run `lym beam status`". That command performs nothing. It shows the re-run only while this machine's own hold stands (card 6's `ThisMachinesHoldNamesTheRerun`).
- If the action failed after the hold cleared or moved, the operator gets no actionable step. Wording such as "run the `lym adopt` re-run `lym beam status` shows" would keep it actionable.

**[LOW] F5's hook is process-global test state the card doesn't name**
- Changed text: card 2's `var holdSetterBeforePushHook func()`, shared by `FixHold` and `ClearHold`.
- Rows that set the hook cannot run in parallel with other `TestWriteAdopt` rows that call the setters. The card doesn't name that constraint.
- Evidence: the testing rule that a test touching process-global state names it.

F12 and F13 (row and `ParseOwners` reuse), and F8, F10 and F11 (wording only): I built no failing case.

Outside my lens: under F4, card 9 now deletes `internal/adoptrecord` at its own batch gate. Integration tests that import it are still outside cards 5, 6 and 9's lists:
- `internal/beamcli/dobbydeps_integration_test.go`
- `internal/beamcli/ownerarchive_integration_test.go`
- `internal/beamcli/status_integration_test.go`
- `internal/orchcli/underadopt_integration_test.go`

If no card edits them, that batch fails to compile.

Tool calls:
- Grepped for `internal/adoptrecord` importers and cards 5–6's targets, to test whether F4's fold leaves an importer at card 9's gate. My context didn't hold the importer list or cards 5–6.
