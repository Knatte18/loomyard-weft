---
round: 1
focus:
  - "Scope against the record: for each deleted test in the range, check it appears in `greedy.tsv` under docs/research/speed/fabric-test-redundancy-2026-11-08/; a drop outside the set, or a test the first cut (`f2ac2bf7`) already handled, is a finding."
  - "Keep rule: tests that pin archive tags and order, deletes on origin, user content never claimed or removed, mutation records and order, or machine-local files kept out of the weft tree should be kept with `//testtiming:keep <reason>`; check the dropped tests in origin, remove_archive, mutation_record, remove_records, gitexclude and weftguards files for such assertions."
  - "Every added `//testtiming:keep` carries a reason that names what it pins; check those added in cleanup_primary, dotlymjunction, prune_dirty, remove_finish, remove_remotetask, resetpairwarp and cleanupremotewarp files."
  - "Production code must not change: confirm the diff touches only test files plus `cmd/lym/cwdmutation_test.go`, and check `internal/fabricengine/export_test.go` for exports removed while a remaining test still needs them."
  - "Leftovers from deletion: unused helpers, fixtures, imports or constants left behind in the heavily cut files (mergein_recovery, warpbinding_clone, commit, merge_target, livestate selftests, clone_adopt), and shared helpers deleted while another file still uses them."
  - "The `cmd/lym/cwdmutation_test.go` allowlist change: check each removed or edited entry matches a test that no longer changes cwd, and that no entry for a surviving cwd-changing test was dropped."
  - "Commit messages and any claim in `_lym/landing/summary.md` about the dropped and kept counts and the keep reasons: check them against the diff."
---

The range is three commits over `f2ac2bf7`.
Most of the diff is deletions, so the questions are about set membership, keeps and leftovers rather than new logic.
