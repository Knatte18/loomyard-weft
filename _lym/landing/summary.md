# Heddle: ship one instruction to settle every edit before the first

Agents in shuttle-launched Claude sessions worked in small steps: a few edits, a gate run, the next failure read, a few edits more, though the card already named what to change.
Edits per edit turn stayed near 2.3 and gate loops ran long.
Nothing asked for all of a change's edits to be settled before the first one.

This change adds that instruction to the read-only heddle layer lym ships, as the entry `lym-plan-edits-before-gate` in `internal/heddleengine/shipped.json`.
It is aimed at the `claude-file` target and renders after `lym-batching` (`order` 20).
It is worded as a command:

1. Read everything the change needs in one message, with a second round only for files the first round named.
2. Decide every edit across every file before making any.
3. Make all edits in one message.
4. Run the gate once, and fix everything its report shows the same way.

It also names what it forbids: one file read per turn, one file read in slices over several turns, an edit/read/edit sequence spread over turns, and a gate run between the edits of one change.

## Design decisions

- The instruction lives in lym's code, not in heddle content, because it is lym's own instruction.
- It is the one home of the read, edit and gate batching rule.
  The batching lines in the stencils are removed so the rule is not stated twice with different wording:
  - `darn-template-write.md` step 1 and step 2
  - `webster-body-implementer.md`
  - `burler-step-1-explore.md`
  - `burler-template-fixer.md`
  - `landing-template-fixer.md`
  - `loom-template-discussion.md`
  - `loom-template-plan.md`
  - `twill-directive.md`
- Stencil text a card still needs is kept: `webster-body-implementer.md` still says the card's named spans are its reads and that its gate runs after its last edit.
  `darn-template-write.md` still says each commit is one change with its own reads, edits and gate run.
- `ShippedBatchingEntryIsPinned` in `internal/heddleengine/store_test.go` becomes `ShippedEntriesArePinned`.
  It pins the shipped layer's slugs, order and target, and the unchanged batching sentence.
  The new entry's text is not pinned, so its wording can be tuned without a test edit.

## To check by hand

- Not run: the effect on agent behavior.
  Check in the transcripts of the first runs this reaches whether agents settle their edits before the first gate run.
  The effect on turns is measured with the rest of `agent-turns`, by note `agent-turns-effect`.
- Confirm the stencils that lost a batching line (the list above) still read well without it, in particular the burler and twill stencils, where the removed sentence stood alone.
