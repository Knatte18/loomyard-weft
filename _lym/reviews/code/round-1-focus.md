---
round: 1
focus:
  - "internal/fabriccli/clone.go, CloneAndWire: the Goal says the cause of the unrendered prime is unconfirmed. Settle whether moving NotifyWorktreeMade after the module-config commit can change what the render reads, or leaves the prime still unrendered on a clone that succeeds."
  - "internal/fabriccli/clone.go: the event now fires after the config commit and before the hub walk. Check every early-return path between the old and new call sites, and the walk's failure path, for a prime left unrendered or a SeamWarning dropped."
  - "internal/fabriccli/clone.go and internal/fabricengine/clone.go: SeamWarning is set on res inside CloneAndWire. Check that every return path after the call carries res, with no `return CloneResult{}, err` after the call wiping the warning, and that the named result and the envelope field `seam_warning` always agree."
  - "The Goal asks that a failed render be printed with its way forward, `lym heddle render` in the prime. Check what text NotifyWorktreeMade returns and whether it names that verb, as the new comments and docs claim, rather than only the log line."
  - "Test coverage: the only new test (worktreeseam_integration_test.go, the failing-seam clone subtest) covers the failure path. Check that no behavior the diff adds is left uncovered, in particular the success path rendering the prime and the printed `seam_warning` field. Judge test economy against the existing seam tests."
  - "Commit b536821a drops the made-event module-config assertion. Check that nothing the Goal asks for lost its only coverage, and that no stale reference to that assertion remains."
  - "internal/hubforge: registerTeardown is exported as RegisterTeardown for a test. Check that the export is licensed by the Goal's scope, that all references to the old name are gone, and that the doc comment opening is updated."
  - "Docs and comments in internal/fabriccli/doc.go, internal/fabricengine/doc.go and internal/hubforge/doc.go: read each changed claim against the code. Check the repository's constraints document for docs that must land with a CLI envelope change, such as a spec or reference for `lym fabric clone` output."
---

Rationale: the diff is small and sits in the clone sequence. The decision record holds only the board entry's Goal,
so the measuring stick is that Goal. The main risks are the ordering change, the failure-path return values,
and the test coverage of the success path.
