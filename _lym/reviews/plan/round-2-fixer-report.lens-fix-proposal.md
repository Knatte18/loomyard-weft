```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F2, F3; F4 was narrowed, and I checked it as well).

no findings

- **F1:** moving the refs to **Edit** matches the Intent text of both cards. Card 1's `docs/overview.md` stays under **Extends** correctly, since it only adds a package-tree line, and `lym loom validate-plan` answers ok.
- **F2:** I found no case where dropping "any lym creel verb" breaks something.
  - `_lym/plan/00-overview.md:43` already says D1 names `lym creel inbox`.
  - `TestLookup`'s row still asserts that name.
  - `inboxCmd` goes through `runVerb` and `client.Call`, so it starts or reaches the daemon.
  - No other card paraphrases the old wording.
- **F3, the neither-word answer:** `bad-answer` agrees with the line that makes the reason at the hour that call's answer, and no text leaks.
- **F3, the refused row:** quoting cuts before escaping, so the id-clash row at line 101 still reaches the notice whole.
- **F3, the non-JSON row:** `exchange` reaches the decoder only for a complete line, and a read failure stays `connect-failed`, so the row reaches `bad-answer`.
- **F3, the `listenHubFake` signature:** the change adds no target, and only cards 7 and 8 use the fake.
- **F4:** "propagated raw" with the narrowed claim holds for `errMissingStateHome`, the only cause the sentence names.

Tool calls:
- One Bash call read the grep of plan paraphrases, the decision record's location and `inboxCmd`. My context lacked `inboxCmd` and any wider search for the way-forward wording.
