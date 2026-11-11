---
round: 1
focus:
  - "Scope against the board entry in the decision record's Goal: the diff touches four test files only. Settle which of the entry's items it covers (root-adopt coverage of an ownerless beam, the hold's skipped steps in `internal/dobbyengine/handover.go`, the owner archive's tip decision rows, an end-to-end archive over a beam's pairs, the records tag against the records branch under `lym adopt --here`, teardown beside an owner archive) and which it leaves out with no `Pattern-Departure:` or recorded reason."
  - "Ownerless-beam tests in `internal/orchcli/underadopt_integration_test.go` and `internal/beamcli/dobbydeps_integration_test.go`: settle whether each builds a beam that truly records no owner, exercises an unfinished root adopt, and asserts coverage through the real `adoptrecord.Watch.Covers` path, so it would fail were the beam read as uncovered."
  - "`internal/fabricengine/remove_archive_integration_test.go`: settle whether it asserts that teardown leaves the owner archive tag alone, still archives the weft tip under its own `archive/` tag, and removes the worktree, and whether its setup could pass vacuously."
  - "`internal/fabricengine/ownerarchive_integration_test.go`: settle whether the added case pins the read against the other side's branch name on the origin, and whether it adds coverage beyond the existing rows of `TestReadOwnerArchives`."
  - "Test economy: check each new test's assertions against existing tests for the same behavior before filing; the redundancy report judged none of them."
---

The diff adds tests only, so correctness findings concern test validity.
The scope question is the likeliest gap, since the entry lists more pins than the four commits' files show.
