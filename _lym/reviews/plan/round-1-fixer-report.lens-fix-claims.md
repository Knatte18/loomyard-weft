```yaml
lens: fix-claims
outcome: reported
```

No findings on the card 1 claims. One finding on card 2.

**MEDIUM: card 2's new doc sentence leaves out the decision halt.**
- **Changed text:** card 2, Docs: "The `InnerRunDeps.HandBack` field doc and `binaryRestep`'s doc name a landing halt or a loom or darn escalate halt as what the seam marks and the re-step leaves alone."
- **Summary:** the sentence says these two docs should name what the seam marks, but the seam also marks a decision halt, and the sentence doesn't name it. The wording came from the reviewer's F3 fix. If the docs are written as told, they will be wrong about the code.
- **Evidence:**
  - `internal/battencli/wire.go:1065` says: "HandBack marks a blocked landing halt or decision halt".
  - `internal/arbiterstop/arbiterstop.go:61-63`: `HandsBack` is true "for a landing halt and for a decision halt". It is `landingshed.HandsBack(...) || strings.HasPrefix(reason, shedadapters.DecisionHaltPrefix)`.
  - Card 2's own `TestRestepSkips` has "a blocked decision halt" answering true, and `restepSkips` wraps `arbiterstop.HandsBack`.
- **Fix:** make the sentence list three halts: a landing halt, a decision halt, or a loom or darn escalate halt.

**Claims checked that hold:**
- **Card 1, line 57:** `loomRerouteCapWayForward` keeps its spent-fixer, over-cap-fixer and Webster plan-conflict causes once the parent-branch cause goes. That matches the constant at `wiring.go:955`.
- **Card 1, line 78:**
  - The `IsRowNeutral` test asserts `parent branch has settled` (`wiring_test.go:1052`).
  - The `TestLoomRerouteCapWayForward` test asserts `for a parent-branch reroute, once the parent branch has settled` (`wiring_test.go:1099`).
  - "in that order" is right.
  - Both doc comments (`wiring_test.go:1048`, `wiring_test.go:1059`) name the parent-branch cause.
  - The second test triggers its reroute with "the fixer's change is over the caps", a cause that survives D1, so it still works after the edit.
- **Card 1, line 59:** the rewritten merge sentence of `Finalize.Call`'s doc and the `mergeIntoTask` doc change match card 1's Intent for `mergeIntoTask`.
- **Card 2, Edit list:** `InnerRunDeps` (`deps.go`) and `innerRunProducer.binaryRestep` (`innerrun_binaryrestep.go:61`) both exist. validate-plan passed.

Tool calls:
1. Read `internal/loomcli/wiring_test.go:1048-1110` to check the line 78 claims against the bodies of both tests. My context held only the two grep lines.
2. Grepped `HandBack` in `internal/battencli` and read `arbiterstop.HandsBack`, to see what the seam marks. My context lacked the closure and the predicate.
