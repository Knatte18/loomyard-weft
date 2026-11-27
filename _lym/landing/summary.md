# Review the Describe text, keep tool terms out, sync the PR body

The change description becomes the pull request's title and body and the landing commit's message, and since PR-Review runs no agent it is the last text a model checks.
This change hardens it in four ways: a mechanical check refuses the tooling's own vocabulary, a reviewer seat checks the text inside the Describe row, the pull request's body follows the description file as GitHub holds it, and a timed-out seat session halts as a resumable stop.

Closes #227
Closes #228
Closes #244

Part of #225 (its first two ideas), #241 (the halt's classification) and #255 (the Describe and writer prompts).

## What changes

**Forbidden terms.**
`summaryparser.ValidateDescription` takes a list of compiled patterns and reports one `forbidden-term` finding per matching line, naming the line number and the matched text.
The patterns come from the new `landing.describe_forbidden_terms` key, a list of Go regular expressions compiled and checked at `LoadConfig`.
The template's defaults cover LoomYard and lym, a Sketch, the decision record, a board entry or note, the scoper, a perch, an orch, "item N of the task" and "the parent recorded".
They are word-bounded, so "lymph" or a lower-case "sketch" pass.
The check reads the description file only, so the credit block and the regression check appended after it are never checked.
It runs in the Describe row's `description` gate, in `lym loom validate-description`, in the beam describer's gate and in `lym beam describe`'s recorders.
The Describe and darn writer prompts state the same rule for the writer.

**Reviewer seat.**
`DescribeTable` adds an assistant seat, `review`, on `landing.describe_review`'s model (default `sonnet[medium]`) for `landing.describe_review_timeout_min` (default 15).
Its prompt, `landing-template-describe-review`, waits for the description, reads the decision record, the board entry, the diff and the run records, then the text.
It reports tool terms, claims the diff or the records do not bear out, missing deliverables, and a wrong Merge danger call.
Its report opens with `# Describe review: approved` or `# Describe review: findings`, which its own gate checks.
The table's gate entry `Describe-Review` stays pending until the reviewer's end marker exists, which holds the writer's turn end.
It then hands a review with findings to the writer once, as a writer-only failure, and passes from then on.
It also passes when the reviewer approved, did not finish or wrote no report, and it always passes at the run's final verdict.
`describeEntry` now puts the row's own `gates` ahead of the table's entries, so the reviewer only reads a text that passed the mechanical check.
An empty `landing.describe_review` keeps the seat from starting.
Its report and marker sit under the never-tracked landing directory (`loomengine.DescribeReviewDir`).

**Pull request body sync.**
`pullrequest.Status` now carries the title and body GitHub holds.
PR-Review's wait used to compare the description's text with the text it had last set itself.
It now compares with GitHub's text at each polled wake, ignoring line-ending and trailing-whitespace differences, and edits when they differ.
A reopened pull request or a body edited by hand on GitHub is therefore set right too.
Finalize does the same before it merges a recorded pull request that is not a beam's.

**Timeout halts.**
In `MultiLLMProducer`, a chair that started and then timed out used to return an error, which halted the run as a producer hard failure.
It now returns Stuck with a reason naming the elapsed time, the limit, the last session state the wait recorded, and `lym loom resume` as the way forward.
The pointer's path is empty, since the output may not exist.
A writer seat's timeout reason carries the same detail.

## Design decisions

- The term list is a config key, not a built-in, because LoomYard's own repository uses these names legitimately.
  A hub whose repository owns them sets `landing.describe_forbidden_terms: []`.
- The reviewer is an assistant seat driven by the writer's gate, not an advisor the writer must remember to ask.
  The seat engine requires the lead's stencil of an assistant table to include `seat-directive-assistant-chair`.
  So the seat is always in the table, and an empty model key skips it rather than dropping it.
- The reviewer's findings go back once per session.
  The "sent" state lives in memory, so after a re-attach the same findings can reach the writer a second time.
- The reviewer waits on the description file itself, so it can read a version the writer is still editing.
  The prompt has it read the description last to narrow that window.
- Not changed here: shuttle's silent hold of an operator-started turn end with outputs missing (#241's sequence).
  With the new classification its timeout halts as a resumable stop rather than a hard failure, but the hold still sends no reminder.
  Also not changed: a Code-Review check of repository documents for tool terms (#255's second item).

## To check by hand

- Before or right after landing, set this hub's `landing.describe_forbidden_terms` to `[]`, after `lym config reconcile --apply` writes the new keys.
  Until then its own Describe rows and `validate-description` refuse descriptions that name lym.
- No test in this change drives the pending review entry through a live shuttle run.
  Watch the first real Describe run: the writer's turn end should wait for the reviewer, and a findings report should arrive as one re-prompt.

## Merge danger

**Door:** two-way
**Blast radius:** landing
