```yaml
lens: fix-proposal
outcome: reported
```

Task: as the fix-proposal lens, challenge the reviewer's proposals that the fixer carried out, by building a case where each proposal is wrong.

Proposals checked: F1, F2, F3, F4, F5, F6 (option 1: confine the release to claims), F7, F8, F9, F10.

1. **MEDIUM**, card 10 (`10-smoke-hook-delivery.md`), new subtest `a Stop claim of more than five lines shows the operator its head once`, changed text "no Stop says line or second Stop hook feedback line carries any post's token".
   - Summary: read literally, the assertion fails on correct behaviour, because the head of a seven-line claim puts posts #2 to #5 on feedback lines two to five.
   - Evidence: decision record lines 36–37 say the summary row shows the joined lines of every Stop hook, clamped to 5, as `Stop hook feedback: &lt;lines&gt;`.
     So the correct output has the second through fifth feedback lines each carrying a post's token.
     "Second Stop hook feedback line" was meant as a second `Stop hook feedback:` row, which is the copy D1 forbids (criterion at line 147, "no second copy").
     As worded, it also reads as line two of the one row.
     The proposal should say "no second `Stop hook feedback:` row, and no `Stop says:` line, carries any post's token".

2. **LOW**, card 4 (`04-ring-claims-mail.md`), the Intent sentence added for F5: "the set a boundary claim would hand the same strand … skipping only the Stop cap and the Stop count".
   - Summary: this holds at card 4 but goes stale at card 7.
   - Evidence: card 7 makes a boundary claim hand out the owed echo first, sharing the 8 KiB budget ("`claimLocked` measures the budget over the echo lines already in `reading` as well").
     A ring hands out no echo (card 9: "a ring hands out none").
     So with an echo owed, a ring's mail set can be larger than the boundary claim's, and the ring also skips the echo, not "only" the Stop cap and count.
     Card 7 does not revise card 4's sentence.

No case found against F1, F2, F3, F4, F6, F7, F8 or F10:
- F1: the newer waiter is rung only after `post "more mail"`, so the claim holds #1 and #2.
- F6: only `Store.Read` and `Store.ClaimAtBoundary` construct a `Reading`, at `verbs.go:264` and `verbs.go:312`.

Tool calls beyond context:
- One grep for `Reading{` constructors under `internal/`, to test F6's claim that every reading `Store.Read` does not make is a claim's; the context had not shown all constructors.
- The same call grepped D1's acceptance and summary wording in `_lym/discussion/decision-record.md`, to test F9's rewording; the context showed only D3's lines.
