```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: rerun the set-premise searches for F1 and F6, and the callers of the changed functions, to see whether the fix missed any member.

- **F1:** after the fix, no non-test file outside `internal/agentname` splits on the machine separator. A search for the literal `"::"` finds only the constant definition and an unrelated `strings.Cut(line, "::")` at `cmd/testtiming/resources.go:355`, which parses test output rather than an agent address. The remaining non-test uses of `MachineAddressSeparator` and `IsMachineAddress` are in `internal/creelengine/verbs.go:181`, `internal/creelengine/crossmachine.go:195` and `internal/orchengine/config.go:136`. They only test for or format the separator and do not split on it.
- **F6:** the old names `maxNameLength`, `maxMachineNameLength` and `machineEnvelopeLead` are gone from the tree. `[lym-bridge]` is defined once, in `internal/creelengine/crossmachine.go:46`. The only other non-test copies are in doc comments, and `bridgeEnvelopePrefix` now derives from `creelengine.MachineEnvelopeLead`. The remaining literal `63`s are in tests and prose, outside the review's scope.
- **Changed signatures:** `noticeReturnedLocked` has exactly two callers, `internal/creelengine/machinemail.go:222` and `internal/creelengine/outbound.go:150`, and both pass the new `receivingMachine` argument. `machineHoldWayForward` has a single caller, `internal/creelengine/outbound.go:164`.

Calls made:
1. `grep` for the literal `"::"` and for `MachineAddressSeparator`/`IsMachineAddress` outside tests and `internal/agentname`, to rerun F1's search and catch splits that avoid the constant name. The context held only a search on the constant name.
2. `grep` for the F6 literals (`[lym-bridge]`, `63`), the old constant names, and the callers of `noticeReturnedLocked` and `machineHoldWayForward`, to confirm no copy or caller was missed. The context held the callers only from before the edit.
