# Gate the arbiter's verdict file so an unreadable one is rewritten

Fixes #754.

An arbiter wrote a `verdict.yaml` whose `verbs` item was a plain scalar holding `: `, so YAML read it as a map, `ReadArbiterVerdict` refused the file and `lyx loom arbiter` turned the verdict into a hand-on with cause `unparseable`.
The arbiter's verbs had already run, so only the resume was lost.

## What changes

- **Verdict gate.**
  The arbiter's run, started or attached, now carries a shuttle gate (`arbiterVerdictGate` in `internal/loomcli/arbiterprompt.go`) that runs `ReadArbiterVerdict` on the attempt's verdict file.
  A file that fails the reader re-prompts the arbiter, with the reader's error and the path to rewrite in the findings file.
  An absent file passes, so a run that ends without a verdict still reads as `missing`.
  The entry is pass-on-cap with a budget of 2, so a verdict still unreadable after that reaches `conclude` and hands on with cause `unparseable` exactly as before.
- **Stencil.**
  `loom-template-arbiter.md` asks for `reason`, `question` and each `verbs` item as a literal block scalar, shows an example, and says an unreadable verdict comes back to the arbiter.
- **Seam.**
  `sheddriver.AgentShuttle`'s `Start` and `AttachIfLive` take a `shuttleengine.GateSpec`; `RunnerShuttle` passes it to `Runner.StartGated` and the new `Runner.AttachIfLiveGated`, the gated form of `AttachIfLive`.
  The tackler passes an empty spec and runs ungated as before.
- **Docs.**
  The arbiter section of `internal/loomcli/doc.go` describes the gate and the block-scalar rule; `internal/shuttleengine/doc.go` names `AttachIfLiveGated`.

## Design decisions

- A gate over the full verdict reader rather than tolerant parsing of `verbs` alone: a `: ` in a plain `reason` or `question` breaks the file the same way, and a ` #` silently truncates a plain scalar, which no parser can recover.
  The gate covers every key and runs whether or not the arbiter remembers to check its own file.
- The tackler's `verdict.yaml` can break the same way and stays ungated here; with the seam taking a gate, gating it is a small follow-up.

## Tests

- `TestArbiterVerb_VerdictGate` captures the gate from both the start and the attach path and checks that it fails the reported verdict shape, naming the parse error, the block-scalar fix and the verdict path, and passes an absent file and a block-scalar verdict.
- `TestArbiterPrompt` now reads the rendered prompt's YAML example through `ReadArbiterVerdict`, so the stencil cannot teach a verdict the reader refuses.
- The existing `unparseable` row of `TestArbiterVerb_VerdictOutcomes` still pins the hand-on once the gate lets a bad file through.
