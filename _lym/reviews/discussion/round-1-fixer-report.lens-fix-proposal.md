```yaml
lens: fix-proposal
outcome: reported
```

Task: run the fix-proposal lens, challenging each proposal the fixer carried out as written.

Findings:

1. **MEDIUM**: decision-record.md D1, the added sentence "This own-name check runs before `machinename.CheckName` judges the machine part, so a post naming any former name, one that fails the name rule included, gets this refusal."
   - Summary: F16's proposal requires splitting the address outside its sole parser, which contradicts D1 and the Constraints.
   - Evidence: D1 says `internal/agentname` "gains its parser, the only place that splits it", and that parser refuses a machine part failing `machinename.CheckName`. The Constraints bullet says "only `internal/agentname` forms, parses or matches the `<machine>::` name". To run the own-name check before CheckName, creel must find the machine part of `Neve_old::ly:orch` without the parser, since the parser refuses that part first.
   - The record does not say where the check lives. One answer is for the parser to take this machine's names and return the own-name refusal itself.

2. **LOW**: decision-record.md D7, "An entry is removed only once creel has answered its settle."
   - Summary: under F3's proposal, a settle that creel refuses keeps its entry forever.
   - Evidence: D3 makes the relay settle "every message its bridge-post record (D7) holds" at each start. "Answered" does not say whether a refusal counts, for example a settle naming an id creel no longer holds as pending. If a refusal does not count, every restart re-sends that settle with no end. The record should say that any answer, a refusal included, removes the entry.

3. **LOW**: decision-record.md D8, the added bullet "A message whose envelope `from` names this machine's current or a former name is skipped with one notice per episode in the bridge channel."
   - Summary: F12's skip has no way forward when a peer machine now uses this machine's former name.
   - Evidence: D6 lets a machine be renamed and approved again by app id, and `machinename.Names` keeps former names indefinitely. Take machine A, renamed from `neve`, and a new peer B named `neve`: every message from B is skipped by A for good. The notice names no step the operator can take, so the bridge between A and B never works.

No further tool calls made.
