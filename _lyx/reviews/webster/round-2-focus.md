---
round: 2
focus:
  - "Check `currentProducer` in `internal/loomcli/start.go`, called from `runDriverSpawnAndWait`: does a status file that fails `state.ReadJSONStrict` (unknown field, truncated JSON) still let `lyx loom start` reach the driver spawn on a clean pair, with the decode failure logged rather than refused?"
  - "Check every refusal `lyx loom start` and `resume` can reach in `internal/loomcli/start.go` (`refuseOverUnfinishedMerge`, `readRecordedParentBranch`, the non-decode error path of `currentProducer`): does each message name an allowed way forward and have a row in `contracts/specs/refusal-spec.md` with a test reaching the production message?"
---

Both entries target the Publish failure and `PATTERN-refusal-way-forward`.
