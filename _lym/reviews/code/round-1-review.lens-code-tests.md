```yaml
lens: code-tests
outcome: reported
```

Task: own the code-tests lens, mapping each behaviour the diff adds or changes to the test that would fail if it broke.

**Per behaviour, the test that pins it**

1. A stored uncaused ruling is judged again once another file joins the range (capture path). Pinned by `TestDriftVerify/…/a later commit that adds the dropped checks to another test has the uncaused ruling judged again` (verify_test.go:1343). It asserts a second judge over both files and a pass.
2. A stored uncaused ruling is reused while the range is unchanged. Pinned by `…/a widened limit with no trailer halts, a trailer-only commit passes, and an unchanged range reuses the ruling` (verify_test.go:1282, the second `f.call()` asserting `f.seats.Calls != 1`).
3. `newRulings` stores `RangeKey` on every new ruling. Pinned indirectly by the same row as item 2: if `RangeKey` were not stored, the stored uncaused ruling's empty range key would never equal the computed one. The resume would then spawn a second judge and fail `f.seats.Calls != 1`.
4. A ruling with no uncaused check is still reused by its file key alone when another file of the range changes. Pinned by: none. The rows at verify_test.go:1282 (final step) and :1449 reuse a caused ruling only over an unchanged range, so their range keys match too. If the `!holdsUncausedCheck(held.Checks) ||` clause at loosened.go:108 were dropped, every ruling would need an equal range key, and no test would fail.
5. The range key changes when the diff of a file already in the range changes, not only when the set of paths changes. Pinned by: none. The new row adds a new path (verify_test.go:1374–1375), so a key that hashed only paths would pass it. Nothing edits the content of a file already in the range while the uncaused file's own diff stays the same.
6. The trailer lines feed into the range key (loosened.go:55, the mutation survivor). No pin is needed, and no test can kill this mutant. The stored ruling must also match the file key (loosened.go:108), and `fileRulingKey` already hashes the trailers (loosened.go:42). So a trailer change always lists the file whatever the range key holds.
7. The range key sorts files by path. Pinned by: none. A test would have to feed the same files in a different order across passes. This only matters if `ChangedFiles` order is unstable; I did not check that, and I raise no finding.
8. The halt names the third way forward, and that a resume judges the files again. Pinned by `requireStuck(…, "commit the dropped checks in a test", "has the judge rule on the range's files again")` at verify_test.go:1354. This is a refusal message the parent acts on, so the assertion is a valid pin, not a static-prose one.
9. Under the recorded `command` diff mode, `passCommand` re-judges an uncaused ruling after the range changes. Pinned by: none at this caller. The shared `listFiles` behaviour is pinned through the capture path (item 1). `passCommand` (verify.go:1240) calls the same `listFiles`, and its early return (verify.go:1243) depends only on `listed.unruled`, so this is no separate behaviour and I raise no finding.
10. An uncaused ruling stored before this change, with no `range_key`, is judged once more. Pinned by: none directly. This follows from item 3's logic (an empty key never equals a computed one), so I raise no finding.

**Findings**

- **MEDIUM, design (test coverage).** Location: `internal/driftengine/loosened.go:108`. No test pins that a ruling with no uncaused check is still reused when another file of the range changes. The `!holdsUncausedCheck(held.Checks) ||` clause can be dropped with no test failing, and the change description (summary.md:14) claims exactly this behaviour. Evidence: verify_test.go:1282–1340 and :1449–1468 reuse a ruling only over an unchanged range, and the new row at :1343–1372 has only an uncaused ruling. Fix: add a row, or an assertion in the new row, where a file ruled `none` or `caused` stays in the range and another file changes. Assert that the judge request does not list that file and that its stored ruling is reused.
- **LOW, design (test coverage).** Location: `internal/driftengine/loosened.go:53`. No test pins that a content change to a file already in the range invalidates a stored uncaused ruling on another file. The new row only adds a new path (verify_test.go:1374–1375), so a range key built from paths alone would pass. Fix: in the new row, keep the second file in the range across both passes and change only its diff.

Tool calls:
- grep of `RangeKey`, `rangeRulingKey`, `holdsUncaused` and the `under command` rows in the driftengine tests, plus a read of verify_test.go:1440–1500. My context lacked whether any test pinned the range key directly and the body of the command-path reuse row.
