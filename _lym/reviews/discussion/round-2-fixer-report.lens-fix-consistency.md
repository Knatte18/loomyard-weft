```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text agrees with its sibling decisions, criteria, ledger rows and cross-references, and looked for any twin left unchanged.

- **LOW**, `decision-record.md` D13 third sentence, unchanged: "A logout removes the file by name …, which in the lym home removes the link, and Prepare creates it again (D5)."
  - Summary: F7's fix changed D5 but left its twin in D13 stale. D13 still says Prepare always restores the link after a logout. It never mentions that a login between the logout and the next Prepare leaves a regular `auth.json`, which D5 now only warns about.
  - Evidence: D5 now says "a regular file or a link elsewhere, as a login in the lym home after a logout leaves (D13), gets a Warn naming the path with the way forward: remove it, and the next Prepare links it again". D5 points at D13 for this case, but D13 describes only the clean relink.

Every other changed passage agrees with its siblings:
- The D1 and D2 marker texts and the `gate failed` copy rule.
- The D1 Bound and the D3 timeout rule, and their criteria.
- The D2 record rewrite and the probe criterion.
- The D5 rules refusal against the D6 Bound, the D6 "What bounds it" line and the D5/D6 criterion.
- The D6 launch line and the Constraints.
- The D12 render change and its criterion.
- Q4, Q5 and Q11, the Open risks and the support log's Rejected alternatives and Round 2.

Further tool calls: none.
