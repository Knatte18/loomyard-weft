# Decision record — arbiter verdict quoting

## Goal

An arbiter's `verdict.yaml` that does not parse as a valid verdict is caught while the arbiter is still live, and the arbiter rewrites it, so a verdict whose verbs already ran is no longer lost to a hand-on with cause `unparseable`.

## Scope

- The arbiter's shuttle run carries a gate that checks the verdict file with the same reader `lyx loom arbiter` concludes with, and re-prompts the arbiter with the reader's error while the file fails.
- The judge-agent shuttle seam in `internal/sheddriver` takes a gate on its start and attach calls, and shuttle gains a gated form of its attach-if-live probe.
- The arbiter stencil tells the arbiter to write every string value of the verdict, each `verbs` item included, as a block scalar.
- The `internal/loomcli` package doc names the gate.

Out of scope: the tackler's own verdict file, which keeps its ungated run.

## Decisions

### D1: a shuttle gate re-prompts the arbiter on an unreadable verdict

Decision: the arbiter's start and its attach to a live arbiter both carry one gate entry whose closure runs `ReadArbiterVerdict` on the attempt's verdict path.
An absent file passes, so the missing-verdict handling stays where it is; a file that does not parse or fails the reader's checks fails with the reader's error and the path to rewrite.
The entry is pass-on-cap with a small re-prompt budget, so a verdict still unreadable after the budget reaches `conclude` and hands on with cause `unparseable` as before.
Rationale: the check runs mechanically whether or not the arbiter remembers to self-check, it covers every key and not only `verbs`, and it reuses the gate the Burler reviewer's review file already uses.
Bound: the gate lets nothing through that `conclude` would refuse; it only adds re-prompts within the arbiter's own deadline.

### D2: the stencil asks for block scalars

Decision: the stencil's verdict section says to write `reason`, `question` and each `verbs` item as a literal block scalar (`|-`), so a `: ` or ` #` inside a command or sentence never changes the YAML's meaning.
Rationale: a block scalar takes any text verbatim, with no escaping rule for the arbiter to get right.

### D3: the judge seam takes a gate

Decision: `sheddriver.AgentShuttle`'s `Start` and `AttachIfLive` take a `shuttleengine.GateSpec`, and `RunnerShuttle` passes it to `Runner.StartGated` and a new `Runner.AttachIfLiveGated`; the tackler passes an empty spec.
Rationale: the seam is the only path the arbiter reaches shuttle through, and an empty spec keeps the tackler's run unchanged.

## Constraints

- PATTERN-shuttle-provider-seam: the gate closure reads only the verdict file and names no provider.
- PATTERN-completion-signal: the gate rides the existing output-file completion path, as the Burler review gate does.
- PATTERN-stencil-ownership: the stencil text changes in `contracts/stencils/loom/loom-template-arbiter.md` only.

## Auto-mode assumptions

- A re-prompt budget of 2 matches the Burler review gate's purpose and is enough for a quoting repair.
- Gating the tackler is a separate change; nothing reported shows its verdict breaking.

## Open risks

- The tackler's `verdict.yaml` can break the same way and stays ungated.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - attach-if-live-gated-removes-nothing-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- The arbiter's start and attach each carry a gate that fails on a verdict file with an unquoted `verbs` item holding `: `, with findings naming the parse error and the verdict path.
- The same gate passes on an absent verdict file and on a valid one.
- A verdict still unparseable once the gate lets the run through still hands on with cause `unparseable`.
- The rendered arbiter prompt tells the arbiter to write string values as block scalars.
