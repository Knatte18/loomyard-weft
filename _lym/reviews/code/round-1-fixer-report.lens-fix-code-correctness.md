```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I read each changed function and test and ran one input through it:

- F6, `missingEngineBlockCopies`: a body that fails `stencil.IncludeNames` now adds one line naming the stencil and the parse error, and the loop continues. The final `sort.Strings` makes the output stable.
- F3, `engine-fork-message`: the Claude copy renders the same text the stencil carried before. The Codex copy reads grammatically. Both copies are registered.
- F1, Webster Master row: Master on `codex`, Recovery on the fake engine. The assertions would fail if the prompt used Recovery's engine.
- F1, Webster verify-fix check: needs "your developer instructions" and rejects "your system prompt".
- F1, begin-batch row: `fx.Deps.Engine = "codex"`; fails with a Claude render.
- F1, twill row: `Values.Engine` is `codex`; fails with a Claude render.
- F1, darn test: the `sol` alias resolves to a Codex lead; a Claude render would fail it.
- Callers: nothing but the test calls `missingEngineBlockCopies`. The `SendMessage` stencil edit leaves the Claude render unchanged.

One LOW observation, outside what the review asked: the doc comment on `missingEngineBlockCopies` (`contracts/stencils/engineterms_test.go`) says it returns one line per missing engine block copy. It now also returns a line for a stencil that fails to parse.

Calls made:
1. `grep` for "as the Agent tool's model" across the tree and for the other `fork-wait` mentions: needed to know whether the `template.yaml` wording or the new blocks are mirrored in a doc or golden file.
