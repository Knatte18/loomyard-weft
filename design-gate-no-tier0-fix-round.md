# lym gate test answers that the hub declares no tier0, leaving a Burler fix round without a gate

`gate-no-tier0-fix-round` · Note · bug, gate, burler

Imported from [issue #237](https://github.com/Knatte18/loomyard/issues/237).

A Burler code fix round ran `lym gate test ./<module path>` on a Go module and the verb answered that the hub declares no tier0.
The round therefore had no gate, and the agent ran `go vet` and `go test` on the module directly.

This is abnormal because the fix-round instructions end each round with the gate; a hub without a declared tier0 leaves the round with nothing to run, and the agent improvised a substitute.

What to do: either have `lym gate test` fall back to a default tier when tier0 is undeclared, or make the refusal name the config key to add, and have the Burler code prompt say what to run when no gate exists.

Provenance: task slug timesheet-suggest-tool; notes burler-code-r1.md.