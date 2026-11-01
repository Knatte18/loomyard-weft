# Heddle: a shipped instruction to settle every edit, then gate once

Webster forks of recent tapestry runs worked in small steps: a few edits, a gate run, the next failure read, a few edits more, though each card already named what to change.
The batching sentence gathers calls whose arguments are known, but nothing asked an agent to settle all of a change's edits before the first one.
This change adds that rule, worded as a command, to every shuttle-launched Claude session's `CLAUDE.local.md`, and makes it the one home of the read, edit and gate batching rule.

## What changes

- `internal/heddleengine/shipped.json` gains the entry `lym-plan-edits-before-gate`, aimed at `claude-file` and ordered after `lym-batching`.
  It orders a change in four steps: read everything in one message, a file whole when it fits and every needed span otherwise, with a second round only for files the first named; decide every edit across every file the card or task touches; make them all in one message; run the gate once and fix its whole report the same way.
  It names what it forbids: one file read per turn, one file read in slices over several turns, an edit, a read and another edit spread over turns, and a gate run between the edits of one change.
- The batching lines `turn-batching` put into the stencils are removed, keeping what is each stencil's own:
  - the darn writer's Step 4 now says each commit is one change with its own reads, edits and gate run, and gates after the last edit of each commit;
  - the Webster implementer's card step says the card is one change, its reads the spans `lym quarry resolve` names and its gate run after its last edit;
  - the burler explore step keeps "your focus, or the whole target", and the burler fixer keeps reading the files the findings name;
  - the Plan and Discussion writers, the landing fixer and the twill lens boilerplate lose their batching sentence outright.
- `store_test.go`'s shipped-layer pin becomes a table over the shipped entries in render order, pinning each slug and its `claude-file` target, and the batching sentence's text.

## Design decisions

- The new entry does not replace `lym-batching`: that sentence covers every call with known arguments, while the new one covers the shape of a change. Both render under `# LoomYard's own instructions`.
- The new entry's text is not pinned by the test, so it can be tuned through `heddle.lym_override` and then copied into `shipped.json` without a test edit.
- Webster's sweep paragraph for retired names ("read every hit's span in one message, then edit every hit in one message") stays: it came with the sweep rule, not with `turn-batching`, and says which reads and edits the sweep is.
- The twill spawn line ("spawn every lens below in one message") stays: it is how the forks start together, not a read or edit rule.

## To check

- The peer task `read-whole-not-slices`, in Finalize, rewrites the same line of `webster-body-implementer.md` this change replaces, and the `batchingSentence` constant this branch's parent already removed from `claudeengine`. Whichever lands second resolves the conflict; its read-whole wording overlaps step 1 of the new entry, so the card line here should win.
- Check the transcripts of the first runs the entry reaches: whether agents settle their edits before the first gate run. Its effect on turns is measured with the rest of `agent-turns` by note `agent-turns-effect`.
- Confirm a freshly rendered `CLAUDE.local.md` shows the new entry after the batching sentence, its numbered list intact.
