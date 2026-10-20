# Gate the arbiter's verdict.yaml so an unparseable one is rewritten

An arbiter that wrote a `verbs` item holding `: ` as a plain scalar produced a `verdict.yaml` that read as a map.
`lyx loom arbiter` then handed on with cause `unparseable`, although the verbs had already run and only the resume was missing.
The verdict is now checked while the arbiter is still live, and the arbiter is asked to rewrite it.

Closes #754

## What changes

- The arbiter's run carries a shuttle gate that runs `ReadArbiterVerdict` on the attempt's verdict path.
  An absent file passes, so missing-verdict handling stays where it was.
  A file that fails to parse or fails the reader's checks re-prompts the arbiter with the reader's error and the path to rewrite.
- Both the arbiter's start and its attach to a live arbiter carry the gate.
- The arbiter stencil (`loom-template-arbiter.md`) tells the arbiter to write `reason`, `question` and each `verbs` item as a literal block scalar (`|-`), with an example, and says an unreadable verdict comes back for a rewrite.
- `internal/loomcli/doc.go` and `internal/shuttleengine/doc.go` name the gate and the new attach form.

## Design decisions

- The gate is pass-on-cap with a budget of 2 re-prompts (`arbiterVerdictGateAttempts`).
  A verdict still unreadable after that reaches `conclude`, which hands on with cause `unparseable` as before, so the gate adds re-prompts within the arbiter's deadline and lets nothing through that `conclude` would refuse.
- The gate checks the whole verdict through the reader, not only `verbs`, and does not depend on the arbiter remembering to self-check.
- `sheddriver.AgentShuttle`'s `Start` and `AttachIfLive` now take a `shuttleengine.GateSpec`, and `RunnerShuttle` passes it to `Runner.StartGated` and the new `Runner.AttachIfLiveGated`.
  `AttachIfLiveGated` is `AttachIfLive` with a gate and, like it, removes nothing on any path.
- The tackler passes an empty spec, so its run is unchanged.
  Its own `verdict.yaml` can break the same way and stays ungated; that is a separate change.
- Block scalars were chosen over quoting because they take any text verbatim, with no escaping rule to get wrong.

## To check by hand

- Not run end to end: a real arbiter session writing an unquoted `verbs` item and being re-prompted.
- Gate coverage on the attach-to-live path was a review finding; confirm the test in `internal/shuttleengine/gate_test.go` and the arbiter attach test in `internal/loomcli/arbiter_test.go` pin it.
