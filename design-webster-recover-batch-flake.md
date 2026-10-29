# Flaky websterengine recover-batch subtest fails once with "record changed"

`webster-recover-batch-flake` · Task · bug, webster

Imported from [issue #902](https://github.com/Knatte18/loomyard/issues/902).

`lym gate test` failed once in a `TestRecoverSpawnOrAttach` subtest of `internal/websterengine` (the one covering a mutating fabric reference at a batch's start on a clean tree), reporting "record changed" although the record it printed was unchanged.
The change under test touches nothing `internal/websterengine` reads, and a rerun of the same gate passed.

This is abnormal: a gate failure unrelated to the change, which passes on rerun, is a flaky test and costs a gate slot and a rerun each time.
The likely cause is in `internal/websterengine/recoverbatch_test.go`: a closure-shared record variable is reassigned by a setup and check pair, and parallel subtests share it, so one subtest's check can compare against another subtest's record.

What to do: give each subtest its own record, by declaring it inside the subtest body or copying it per iteration, and confirm with repeated runs under the race detector.

Provenance: task `shuttle-hook-signals`, from note `Darn-Write.md`.
