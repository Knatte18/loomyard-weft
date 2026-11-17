```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I tested each reviewer proposal the fixer carried out by building a case where the proposed change goes wrong.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F10, F11, F12, F13, F14. I also challenged F9, where the review left the choice to the fixer and the fixer chose to let the arbiter settle a stalemate.

1. **MEDIUM**
   - Changed text: D4, "In an incremental round the entry refuses any other `origin: parent` finding".
   - Summary: the refusal also catches a carried parent finding, so a still-open parent finding cannot be carried into the next round.
   - Case: round 1 has a parent ask, and finding F2 is tagged `origin: parent` with `parent_ids: [P1]`. The fixer disputes F2, and the round 2 reviewer keeps it `open` and carries it, with its origin. Round 2 holds no ask, so the entry refuses the carried finding. Yet D3 requires every `open` entry to be carried as exactly one finding. The two rules conflict until the gate passes on its cap.
   - Evidence: D3, "Each `open` entry is carried as exactly one finding". The exemption and the refusal are stated for any `origin: parent` finding, not only for new ones.

2. **MEDIUM**
   - Changed text: D3, "keeps its earlier finding's location path … refuses a carried finding that … changes its location path".
   - Summary: a defect that the fix moved to another file can no longer be carried. Unless it is BLOCKING, it cannot be raised as new either, so it is lost.
   - Case: round 1 has a MEDIUM finding on `a.go`. The fixer marks it `fixed-differently` by moving the function to `b.go`, but the defect remains. Carrying it with path `b.go` is refused. Carrying it on `a.go` points at a file that no longer holds the defect. Raising it as new on `b.go` is refused because it is not BLOCKING. The reviewer's only legal choice is `outdated`, which drops a live defect.
   - Evidence: D4, "the entry refuses it unless it is `BLOCKING`"; D3, "`outdated` means the code … is gone or rewritten".

3. **MEDIUM**
   - Changed text: D1, "The fix check passes on its attempt cap … A report whose frontmatter then fails to parse leaves its round incomplete …".
   - Summary: the cap behaviour only covers a report that does not parse. A report that parses but fails the check's other refusals also passes on the cap, and nothing says how D2 renders it.
   - Case: after the cap, a report parses but omits F3's disposition, or holds a `disputed` entry with no `rebuttal`. D2 counts the round as complete and must render F3 with no disposition. D3's "`withdrawn` is legal only for a finding round N-1's fixer disputed" then judges against a malformed entry.
   - Evidence: D1 lists the refusals "miss, repeat or invent a finding id, or that carries `spawned`". D2's completeness test is "frontmatter parsing".

4. **MEDIUM**
   - Changed text: D6, "`internal/arbiterstop` admits `stalemate` … the arbiter settles a stalemate escalation", and the Bound, "an `accept` from the parent or the arbiter".
   - Summary: letting the arbiter accept a stalemate contradicts the unchanged Constraints entry that classifies `review_stalemate_rounds` as `CategoryNone`.
   - Case: an operator lowers `review_stalemate_rounds` to 1. The arbiter can then `accept` a segment with open findings one round after the first unchanged round. The key now changes when a segment passes without the parent, which "lets no gate pass" no longer describes.
   - Evidence: Constraints, "`PATTERN-config-layers`: … its value only moves an escalation to the parent earlier or later and lets no gate pass".

5. **LOW**
   - Changed text: D2, "the parent's recorded answer: round N-1's settled decision file, written through the `lym loom circling` verbs".
   - Summary: a decision file of any cause is rendered as the parent's answer to a `decision`-class finding, so D3 can resolve a question the parent never answered.
   - Case: round N-1 escalated with cause `circling` or `stalemate`, and the parent answered `continue`. D2 renders that continue beside each held `decision`-class finding. The reviewer marks the finding `resolved` and does not carry it, and `routeDecisions` never asks the parent the question.
   - Evidence: D3, "A `decision`-class finding the parent answered, as D2's block shows, takes `resolved` … and is not carried". D2 does not limit the decision file to cause `decision`.

Further tool calls: none.
