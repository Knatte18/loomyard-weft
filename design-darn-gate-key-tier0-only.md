# gate test: darn-write gate key runs tier0 only, so a tier3 measurement test runs nothing

`darn-gate-key-tier0-only` · Note · bug, gate, darn

Imported from [issue #233](https://github.com/Knatte18/loomyard/issues/233).

A prompt told an agent to run a measurement test through `lym gate test --gate darn-write`. That gate key runs tier0 only, while the test lived in tier3 (a slow-test project, category `ModelsLong`). The first call ran no test, and the orch had to allow `--tier 3` by hand.

This is abnormal because the prompt and the gate key disagree on the tier. Either the prompt generator should pass `--tier` when the named test is not tier0, or the gate should say, when it runs no test, that the filter matches nothing at this tier and name the tiers that would match.

Provenance: task models-stick-slip-floor-tune, note Darn-Write.md.