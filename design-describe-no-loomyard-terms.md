# The Describe text names no LoomYard terms, except the Created using block

`describe-no-loomyard-terms` · Note · enhancement, landing, stencil

Imported from [issue #227](https://github.com/Knatte18/loomyard/issues/227).

## Seen (operator, 2026-11-24)

An analysis of the parent's PR-Review findings in the C# hubs found that about 12 findings were internal process or tool names in the PR text: "item 3 of the task", "the test scoper cannot run C# tests there", "the task's branch point", "The board entry proposed", "The parent recorded...".
The stencil that writes the description (`landing-template-describe`) does not forbid them, so the parent had to tell the Describe session so in each hub.
In one run the parent noted its own earlier instruction had caused it.

The operator's rule: LoomYard, and anything from it, does not belong in the Describe text.
The one exception is the "Created using" block at the bottom.

## Idea

Keep LoomYard's vocabulary out of the description the Describe row writes, and let lym hold it.

- Add the rule to the Describe stencil: the description names no board, task, Sketch, decision record, parent, orch, scoper, gate, tier, perch or lym term, and no "item N of the task".
- Let the Describe row's mechanical gate (`NewDescriptionGate`, which runs `summaryparser.ValidateDescription`) refuse such a term outside the credit block, with the refusal naming the term and the line.
- Leave the credit block as it is: the `credit` keys in `landing.yaml` append it, and the term check skips it.
- Read first how the credit block is appended to the description, and which words the check can match without refusing a repository's own legitimate names.
- The issue on reviewing the Describe text covers claims and missing content; this issue covers the vocabulary.
