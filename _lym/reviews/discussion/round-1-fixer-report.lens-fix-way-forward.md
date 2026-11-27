```yaml
lens: fix-way-forward
outcome: reported
```

Every changed way forward was checked: D4's hold, D5's no-wait option and cap refusal, D6's hand-on and segment continue count, D7's relay rules and pane relay, D8's dedup, D10's three chases, D11's split probe and D12's warning. Two of them cannot be carried out or lead nowhere. The rest hold: D6's per-segment count can be read because each segment keeps its own run directory, which `internal/shedadapters/burlerfix.go:322` describes.

**MEDIUM: the orch cannot name a halt by its history entry**
- **Changed text:** D7, "Its relay names the halt by its row and history entry, as the dobby's post does". D8's dedup relies on it: "for the same slug and the same halt, its row and history entry".
- **Problem:** The notice the orch reads does not carry the history entry, so the orch cannot write its relay that way. A relay and a reply about one halt would then not match the dobby's post in D8's dedup.
- **Evidence:**
  - D7 adds only `class:`, `settle:` and `halts_at_row:` to the hand-back notice, beside `autonomy:`.
  - `internal/orchcli/notify.go:94-98` (`WithRunAutonomy`) appends only `; autonomy: …`.
  - Nothing in the changed text adds the history entry's index to the notice.
  - The orch-classified severe halts in D7's relay list, a scope change and a cancel, may not sit at a halt entry at all.

**MEDIUM: a held run whose pause is lifted is left with no way forward**
- **Changed text:** D4, "A run whose pause the orch set as a hold (D6, D9) is never resumed by the lift … only the orch or the operator resumes such a run".
- **Problem:**
  - The review's F11 case is a blanket `lym tender resume`. It lifts the hold's pause entry too.
  - After that, D4 leaves the run halted at a pause hand-back that nothing chases.
  - D10's chase covers a halt entry reading `settle: orch`, but the pause row's verdict is `lym`.
  - D10 also counts "a pause covering the run" as ending the chase.
  - So the run sits until someone notices it, which is the overnight stall D10 exists to prevent.
- **Missing:** No changed text says the hold survives a blanket resume, or that a lifted hold is chased or posted.

Tool calls:
- I grepped `internal/shedadapters` for "segment" to check that the arbiter can tell which circling decision files belong to its segment; my context did not hold how segments are laid out.
- I listed the Go files outside tests that write `autonomy:` to find the code that builds the hand-back notice; my context did not hold it.
- I grepped `internal/orchcli/notify.go` to see whether the notice names a history entry; my context did not hold the notice's format.
