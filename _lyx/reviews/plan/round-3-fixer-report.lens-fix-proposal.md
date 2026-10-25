```yaml
lens: fix-proposal
outcome: reported
```

Task: test whether the reviewer's proposals that the fixer carried out (F2–F5) are themselves right.

Findings:

1. LOW — card 17's owner refusal, the clause starting "`lyx adopt <slug> --keep-local` here, which records this machine as owner over the stopped weft".
   **Summary:** the refusal now tells the operator to run `--keep-local` every time, which skips the choice adopt offers when it stops.
   **Evidence:** `--keep-local` only matters when adopt stops; `adopt.go:334` checks `!req.KeepLocal`.
   When adopt stops, its message at `adopt.go:356` offers both ways forward: `--keep-local` keeps the local commits, and `--drop-local` archives them and resets to origin.
   But the owner refusal also fires when the weft is not `anchor_stopped`: B can differ from the upstream marker while the checked-out anchor and upstream agree, or the weft can have no upstream.
   In that case a plain `lyx adopt <slug>` goes through.
   Prescribing `--keep-local` there means a diverged weft never shows its stop with both choices, so the operator never gets the `--drop-local` option.
   A better wording: plain `lyx adopt <slug>` here, with `--keep-local` when it stops over an `anchor_stopped` weft.

2. LOW — card 17, the prime part of the same clause: "for the prime `lyx adopt --prime`, which refuses over an `anchor_stopped` prime weft naming the `--set` to the upstream tip's marker".
   **Summary:** when the prime weft is stopped, this step leads straight back to a way forward the same refusal already lists.
   **Evidence:** the clause ends "or, when the weft has an upstream …, `lyx fabric anchor <slug> --set <the marker at the fetched upstream tip>`, which this refusal lets through".
   In the stopped case, `adopt --prime` only refuses and names that same `--set`, so the operator takes an extra hop to reach it.
   It is not wrong, just a redundant step.

3. LOW — card 8, the added sentence naming "inside the write callback of `Bolt.CommitWritten`". The fixer pinned this method; the reviewer wrote "the `Bolt` commit's write callback".
   **Summary:** the deletion itself is correct, but `CommitWritten` does not pull before it commits.
   **Evidence:** `bolt.go:40-49` says `CommitWritten` "never pushes" and does no pull first, while `PullThenCommitWritten` (`bolt.go:183`) is the sibling for "callers that must not write over a stale copy".
   The card says "one board commit and push through `Bolt`".
   Over a stale Bolt that push can be rejected, and naming `CommitWritten` now rules out the pull-first path.
   Its contract does stage the deletion correctly ("A path that no longer exists stages its deletion"), and `root.Remove(` matches none of the guard's banned tokens.

F3 rows (pass wait, sink close, sink reopen) and F5's caller list: no case found where they are wrong.
The sink-reopen row matches the Intent: `Apply` ran, so `Close` leaves the record and the error names the re-run.
Card 15:61 confirms `Close` calls `RemoveMoveRecord`.

Tool calls:
- Read `internal/fabricengine/bolt.go` (CommitWritten, PullThenCommitWritten) and `cmd/lyx/destructiveguard_test.go`'s banned tokens: my context did not hold `CommitWritten`'s contract or the full token list.
