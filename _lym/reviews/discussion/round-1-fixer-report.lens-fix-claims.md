```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked each claim the fix added or rewrote against the code at HEAD.

**MEDIUM: D4's rule for finding the block's end breaks after a goto out of the block**
- Changed text: D4, "The end has no history entry of its own; a reader finds it as the exit's first entry after the block row's, the first entry whose block path does not include the block."
- Problem: a goto from inside a block to a row before the block puts an entry with no block path into history before the exit's entry. That row could be Preflight, or the block row itself. D4's rule would read that goto entry as the block's end.
- Evidence: `internal/shedengine/goto.go:338-341`, in `gotoAdmitted`, admits every row before the reference row in list order, and the block row and Preflight sit before the inlined rows. `goto.go:136` appends a `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto}`, and `goto.go:250` does the same on re-launch. D6 says goto and relaunch entries take the block path of their target, so an entry for a target outside the block has none. That makes the two halves of the sentence disagree: "the exit's first entry" and "the first entry whose block path does not include the block" name different entries.

These claims hold:
- D3: only a goto or a re-launch bypasses the block gate. The only writes of the entered row are `goto.go:124` and `entry.go:97` (the gate held). `run.go:903` and `goto.go:244` clear it, and `rename.go:62` only renames it.
- D3: both moves need a halted run and a row `CheckGotoTarget` admits. `Goto` refuses a running or done run (`goto.go:96`, `:107`), `Relaunch` does the same (`:223`, `:226`), and both call `CheckGotoTarget` (`:110`, `:230`).
- D3: a re-launch onto the block row clears the entered row and evaluates the gate (`goto.go:244`).
- D3, "every reader that reconstructs where a skipped entry routed": `goto.go:317-318` is the only such reader. The other readers of `OutcomeSkipped` (`run.go:647`, `activity.go:30`, `loomcli/checklist.go`, `settledguard.go`, `darnwiring.go`, `timingspans.go`, `samplerengine/record.go`) never work out where the skip routed.
- The plan-writer note's claim that the goto admission reads a skipped route from `OnDone` matches `goto.go:317-318`.
- D4: a skip routes by `OnDone` (`entry.go:52-70`).
- D8: profile blocks carry review budget keys (`internal/runprofile/template.yaml:18`, `:27`, `:37`, `:47`).

Tool calls:
- Grepped the writers of `EnteredRow`, the readers of `OutcomeSkipped` and the `Outcome` constants: my context did not have the full list of readers and writers needed to test the D3 claims.
- Read `run.go` lines 630-659: needed to confirm that the `OutcomeSkipped` reader at `run.go:647` does not work out a route.
