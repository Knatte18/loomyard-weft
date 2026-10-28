# Testing the shuttle hook-signal change: an unrelated websterengine subtest failed once

The first `lym gate test --wait` failed in `TestRecoverSpawnOrAttach/a_mutating_fabric_reference_at_the_batch's_start_on_a_clean_tree_is_refused_toward_the_accept-audit_route` with "record changed", although the record it printed was unchanged and the change touches nothing websterengine reads.
A rerun passed.
The likely cause is the closure-shared `rec` in `internal/websterengine/recoverbatch_test.go`: a setup and check pair reassigns it, and parallel subtests share it, so one subtest's check can compare against another's record.
