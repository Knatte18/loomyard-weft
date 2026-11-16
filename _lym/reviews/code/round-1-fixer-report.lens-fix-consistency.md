```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the F1 change set, which is the new filing-mark subtest in `internal/battencli/beamland_integration_test.go`.

Findings:

- **LOW**, file header of `internal/battencli/beamland_integration_test.go`, lines 1-4.
  - The header lists what the file covers: start guard and skip, unreadable-pair refusal, promote refusal, sub-beam and landing refusal.
  - The new subtest `GroupingProjectWhoseRunFiledItsFamilyResumesOnlyWithTheFilingMark` is a filing-mark resume case and fits none of those lines.
  - The header is now incomplete. A reader deciding whether this file covers the filing-mark wiring would not learn that it does.
  - Suggested fix: add a line such as "It also covers a filed run resuming under its family's grouping project."

I found no other inconsistency:

- The write location matches the read. The test calls `shedrun.WriteFilingMark(h.Location, "g")`, and the hub guard reads `shedrun.FilingMarked(location, slug)` with the same location and slug.
- The mark stays on disk for the later subtests. The later subtests (`MemberHolds...`, `BeamWithMembers...`) use slugs `outer`, `c` and `t` and never read the mark for `g`.
- The only non-test reader of the mark outside the guard is `internal/loomcli/wiring.go:280`. It reads from the prime under a resolved run ID, and no other check in this test touches it.
- The seeding comment at lines 37-40 still holds. `g` is the grouping project under `outer`, and it is not a beam, so it reaches `refuseProject`.

Tool calls:
1. Grepped `FilingMarked|FilingMarkFile` across the non-test Go code, to confirm no other reader of the mark could be affected by it staying on disk. My context did not hold that search.
