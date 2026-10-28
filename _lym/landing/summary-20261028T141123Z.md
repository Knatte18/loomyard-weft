# Stencil and system-prompt sentences that batch independent calls

The turn audit in `docs/research/turn-audit-2026-10-28/report.md` found that the heavy roles spend many turns on single tool calls that did not depend on each other, and on edits made one per turn.
This change adds the audit's section 5 sentences to each heavy role's stencil, and one batching sentence to the system prompt every Claude session shuttle launches with a run directory gets.

## Stencils

Each sentence sits at the step where the role first reads or edits:

- `darn/darn-template-write.md`, Step 4: read every file the change touches, with its greps, in one message before the first edit, make independent edits in one message, and run `lym gate test --wait` once after the last edit of each commit instead of testing as it works.
- `webster/webster-body-implementer.md`, step 2 of the per-card sequence: read the spans `lym quarry resolve` names in one message, batch the card's independent edits, and run the card's gate once after its last edit.
- `loom/loom-template-plan.md` and `loom/loom-template-discussion.md`, Step 2, and `landing/landing-template-fixer.md`, after `## What to read`: issue every search and read you can name in one message; wait for a result only to choose the next call.
- `twill/twill-directive.md`, the fork boilerplate every lens fork copies: the same sentence for the fork's further calls.
- `burler/burler-step-1-explore.md`: read the files of the focus, or of the whole target when no focus is set, in one message, then check them.
- `burler/burler-template-fixer.md`: read the findings' files together, and apply the corrections to one file in one message.

The burler stencils serve the Discussion, Plan and Code rounds alike, so the code-review rows of the report are covered by the same two sentences.

## Appended system prompt

`internal/shuttleengine/claudeengine` now writes the deny notice, the creel trust sentence and a new `batchingSentence` into `append-system-prompt.txt`, space-separated, with the notice left out when no deny is installed.
The sentence reads: "Request every call whose arguments you already know in one message, and read what you need before you edit; wait for a result only to choose the next call."
The file already rides `--append-system-prompt-file` on the launch and the resume line, so the sentence survives a resume.
The package's init check now refuses a line break in either sentence.
The Prepare and settings tests pin the new file contents, and the package doc and `docs/overview.md` describe them.

## Design decisions

- The stencils say "message" where the report says "turn": one message carries several tool calls, and the Edit tool applies several edits when they arrive together, while a turn can span several messages.
- The darn gate runs once per commit rather than once per session, since the stencil keeps its rule that each commit leaves the tree green.
- Webster's per-card gate and commit stay mandatory; the sentence only removes repeated gate runs within a card.
- A session opened with `lym shuttle open` writes no run directory, so it gets neither the creel sentence nor the batching sentence, as before.

## Measurement

No bench ran before the change.
The next loom and darn runs after landing are to be measured against the report's figures by re-running its scripts on their transcripts.
