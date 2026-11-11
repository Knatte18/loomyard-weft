```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside its own lines.

**Per change, the set I found:**

- **Ownerless-beam watcher test** (`internal/orchcli/underadopt_integration_test.go`): the diff changes no production code.
  - File header: covered. It was extended to name the new beam case.
  - Callers and readers of the code under test (`beamOwnerElsewhere`, `underAdoptWatch`, `Watch.Covers`): unchanged.
  - Docs and stencils: none describe it.
  - PATTERN: the new test follows `PATTERN-test-speed` (`tiergate.Require(t, 1)`, `t.Parallel()` first) and `PATTERN-test-isolation` (`hubforge`).
  - Covered.
- **Ownerless-beam dobby test** (`internal/beamcli/dobbydeps_integration_test.go`): no production change.
  - The file header still reads "covers the seams that differ between a beam's dobby and the prime's, and the seams it fills for both". That still describes the new test, which pins the `Owner` and `UnderAdopt` seams of a beam's dobby. Covered.
  - The `adoptrecord` import is added and used.
  - PATTERN fit is the same as above. Covered.
- **Decoy rows in `TestReadOwnerArchives`** (`internal/fabricengine/ownerarchive_integration_test.go`): the `decoyBranch` field is documented. The file header ("covers `ArchiveOwnerTips` and `ReadOwnerArchives`") still holds.
  - `TestReadOwnerArchives_ListsEveryPairSortedBySlugSideAndTag` shares no table with it and is unaffected.
  - Covered.
- **Teardown test beside an owner archive** (`internal/fabricengine/remove_archive_integration_test.go`): the file header gains the matching line.
  - The helpers `archiveTagsAt` and `tagTargetAt` are reused, not duplicated.
  - The `archive/*` list pattern cannot match the `archive-owner/` tag, so the "exactly one archive tag" assertion is sound.
  - New imports (`fs`, `slices`, `lymcwd`) are used.
  - The `//testtiming:keep` marker is not needed: the redundancy report says "unclassifiable call", not contained.
  - Covered.

**Findings:** none. No signature, output, message or file shape changes, so no caller, doc, stencil, fixture or PATTERN entry is left wrong.
- The commits carry no `Pattern-Departure:` trailer, and none is needed.
- The diff touches no `PATTERN.md` or `pattern/` file, and I found no declared PATTERN changes in the directive.
- No stencil changed.

**Tool calls:**
1. `head` of the two integration test files plus a grep for `t.Parallel`, `Chdir` and `Setenv` in the orchcli test. I needed the unchanged file headers and the parallelism of the new test, which my context lacked.
