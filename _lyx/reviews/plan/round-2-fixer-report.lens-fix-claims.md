```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: checking that each claim the fix added or rewrote holds against the code at HEAD.

no findings

Tool calls:
- Read `teardown_integration_test.go`'s build tag, the two InnerRun rows of `stuck_test.go`, the production `Sleep` readers in battenshed and `envkit`'s Sleep seams. My context did not show the tier2 tag, the row bodies' hooks, the full list of non-InnerRun `Sleep` readers (to check "nothing in the producer sleeps" and "`waitOrCancel` stays for Land's and the row notice's defaults") or `envkit`'s Sleep entries.
