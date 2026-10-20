# Gate the arbiter's verdict.yaml so an unreadable one is rewritten live

An arbiter that wrote a `verdict.yaml` the reader rejects used to end its turn and lose the verdict.
A `verbs` item such as `lyx loom decision add ... --title "D6 refined: card 9's ..."` left unquoted reads as a YAML map, so the verbs had already run when the driver handed on with cause `unparseable`, and only the resume was missing.
Now the file is checked while the arbiter is still live, and the arbiter is re-prompted with the reader's error to rewrite it.

Closes #754

## What changes

- `internal/loomcli`: `arbiterVerdictGate` returns a `GateSpec` with one `verdict` entry that runs `loomengine.ReadArbiterVerdict` on the attempt's verdict path.
  An absent file passes, so the missing-verdict handling stays where it was.
  A file that does not read fails, and the findings carry the reader's error and the path to rewrite.
  `arbiter.go` passes the gate on both the fresh start and the attach to a live arbiter.
- The arbiter stencil (`contracts/stencils/loom/loom-template-arbiter.md`) tells the arbiter to write `reason`, `question` and each `verbs` item as a literal block scalar (`|-`), with an example, and that an unreadable verdict comes back for a rewrite.
- `internal/sheddriver`: `AgentShuttle.Start` and `AttachIfLive` take a `shuttleengine.GateSpec`; the tackler passes an empty one, so its run is unchanged.
- `internal/shuttleengine`: new `Runner.AttachIfLiveGated`, the gated form of `AttachIfLive`.
- The `internal/loomcli` and `internal/shuttleengine` package docs name the gate and the new method.

## Design decisions

- The gate checks the whole verdict with the reader `lyx loom arbiter` concludes with, so it covers every key and does not rely on the arbiter remembering to self-check.
- The entry is pass-on-cap with 2 re-prompts.
  A verdict still unreadable after that reaches `conclude` and hands on with cause `unparseable` as before, so the gate lets nothing through that `conclude` would refuse.
- Block scalars were chosen over quoting because they take any text verbatim, with no escaping rule to get right.
- `AttachIfLiveGated` removes no superseded strand on any path, like `AttachIfLive`; a test pins this.
- The gate closure reads only the verdict file and names no provider.

## Not done

- The tackler's own `verdict.yaml` can break the same way and stays ungated.

## Check by hand

- Nothing run beyond the unit tests; no live arbiter was driven against a deliberately broken verdict.
