---
round: 2
focus:
  - "Card 6 (`_lyx/plan/06-loom-fmt-verb.md`): `lyx loom fmt <file>...` goes through loom's pre-run (`armAt`, `resolveRunID`, `resolveRecipe` in `internal/loomcli/arm.go`), and `shedrun.ReadSeed` refuses a run-id containing a path separator. Settle whether the card makes a path argument with a separator reach `fmtCmd`'s `RunE`, and whether a test exercises that pre-run path rather than calling `fmtCmd` directly."
  - "Card 6: `planGoFormat` runs `filepath.EvalSymlinks` before its inside-root and `.go` checks. Settle whether every refusal reason the card lists, including a nonexistent path and an unreadable file, has a named input and a test row that reaches it."
  - "Card 7 (`_lyx/plan/07-worktree-leftover-retry.md`): it adds `removeWorktreeLeftover(rec *Mutations, ...)` to `internal/fabricengine/destroy.go`, and `destructiveGuardRecordingExecutors` in `cmd/lyx/destructiveguard_test.go` is the table `TestMutationRecord_FabricengineProductionSource` checks. Settle whether the card pins the new executor's leading recorder per `PATTERN-mutation-record`."
---

Any fix applied to cards 6 and 7 is new text; read those cards as written now.
