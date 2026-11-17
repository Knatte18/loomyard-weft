```yaml
lens: fix-way-forward
outcome: reported
```

Task: check every way forward that the fix added to the decision record (refusals, outcomes, criteria) and confirm each can be reached and leads somewhere useful.

no findings

- F1: the failed-listing check reads HEAD and the files the package itself owns. Neither needs the linked set, so the check can run in the state the decision describes. A failed check leads to the existing D6 path: the package is listed as unmeasured, the dirty files are named, and the call exits non-zero. D9's way forward from that state is to commit or revert, then rerun `lym gate test --map <pkg>`, which makes progress. The new D6 criterion puts the edit in the package's own file, so a test can reach it.
- F2: a build or test past its bound leads to the outcomes D7 and D3/D5 already handle (a failed build, or a timed-out test that is rerun alone). There is no new dead end.
- F6: the new precondition on the D7 criterion ("no green record from before the map run") is easy to set up in a test, because a fresh tree has no green record.

One thing outside this lens: if an uncommitted edit in a *linked* package makes the listing fail, the check of the package's own files passes. That failure is still written under HEAD, so the new F1 sentence narrows the gap without closing it.

No further tool calls made.
