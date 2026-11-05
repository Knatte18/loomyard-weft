```yaml
lens: fix-way-forward
outcome: reported
```

The fix-way-forward lens found three LOW findings, below. Every other way forward the fix added can be reached from the state it names and leads to progress.

- **LOW**: D9's opening paragraph, "in interactive mode matches the proposal's hash against an approval recorded after the latest return (D7, D8)".
  - The fix names the hash check as a precondition of filing but gives no outcome when it fails.
  - The D6 check next to it does get one: "a failing D6 check returns to Raddle-Write as a D8 failure does and counts among its returns".
  - The D7 criterion says only that "Raddle-File refuses a proposal edited after approval". Neither text says whether that refusal is a Stuck return, whether it counts among the 2 returns, or whether it is a halt. So the run has no named next step when `proposal.yaml` is edited between `raddle-approve` and the session's end.
- **LOW**: the same D9 paragraph, "a failing D6 check returns to Raddle-Write as a D8 failure does".
  - A D8 return is defined as Stuck back to Raddle-Write "with the moved entries named".
  - Carried over word for word, the new session for a D6 failure would be told about moved entries rather than the D6 findings it needs to revise the proposal.
  - The text should say that the return names the failing checks.
- **LOW**: D2's new rule, which keys on "a note flagged `discussed` with a stored record", with the way forward `lym board promote &lt;slug&gt; --recipe raddle`.
  - The discussed flag is set by Discussion's record store on the entry under discussion (`internal/loomcli/discussionrecord.go:29` → `StoreDecisionRecord`), and a merge keeps it (`internal/boardengine/store.go:1122-1137`). It is not set only on Sketches.
  - So a discussed task that is made a note again, without becoming a Sketch, can afterwards be promoted only through a raddle run. The refusal's way forward then sends it through a split proposal when going back to tapestry is what's wanted.
  - I did not confirm that a task-to-note write on a discussed entry is reachable today. If it is, D2 should name that case, or say it is intended.

Checked and sound:
- **Halt after step 3:** `lym loom resume` reads shed run state, not the board (`internal/loomcli/entryobservation.go:96-118`, `resume.go:67`), and dobby uses a done entry only for the land notice (`internal/dobbyengine/landnotice.go:105`). So the parent can resume a run halted after step 3, and D9's step-3-done marker skips to step 4.
- **D2's refusal:** `promote --recipe raddle` restores the dropped label, so the way forward can be carried out.
- **D8's cleared approval:** it leads to a new interactive session that must run `raddle-approve` again before the row can reach Raddle-File, which is reachable.

Tool calls:
1. grep of battencli and loomcli for done handling and resume functions: my context did not show how a halted run is resumed.
2. grep of `loomcli/resume.go` and `entryobservation.go` for board reads: to see whether resume reads board status.
3. Read of `entryobservation.go:60-119`: to confirm that resume observes shed state only.
4. grep of battencli and dobby for done-entry handling: to see whether a done entry with a live run gets torn down.
5. Read of `store.go` `Promote` and grep for where the discussed flag is set: to see which entries can be discussed notes, for D2's new key.
6. Read of `store.go:1110-1145` and grep for `StoreDecisionRecord` callers: to confirm the discussed-flag writers beyond Sketches.
