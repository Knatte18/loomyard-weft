# Agents read a file whole or all its spans in one message

Webster implementers read one file in three to five slices over as many turns, and re-read the same files in later turns.
Each extra turn re-pays the session's context, and about 80 of the five heaviest forks' 208 single-call read turns could have been issued a turn earlier, since the path or line range was already known.
This change tells agents to read a file once, in one message.

## What changes

- The batching sentence in `internal/shuttleengine/claudeengine/settings.go` (`batchingSentence`), which every Claude session shuttle launches receives through `append-system-prompt.txt`, now also asks for a file to be read whole, or every span needed from it, in one message.
  It asks the session to keep a file it has read in mind rather than read it again, unless something other than its own edit has changed it.
- The Webster implementer stencil, `contracts/stencils/webster/webster-body-implementer.md`, asks the same of the fork before the card's first edit: read in one message every file the card needs, a file whole when most of it is needed, otherwise every span, never one file in slices over several turns.
  A file read in that batch is read again only after a change not made by hand, such as a rename tool's run, and never to check the fork's own edit, which the Edit tool confirms.
- `docs/overview.md` (shuttle paragraph) and the `claudeengine` package documentation (`doc.go`) describe the extra ask.

## Design decisions

- The re-read exception is worded as "something other than your own edit": a rename tool or any outside change still justifies a re-read, while checking one's own edit does not.
- The stencil's earlier "instead of ... reading whole files" is dropped from the `quarry resolve` line, since the new sentence now says when a whole file is right.
- The stencil and the batching sentence carry the same rule at two levels: the sentence reaches every agent, and the stencil gives the implementer the card-specific form.

## To check by hand

- Nothing was run against a live Webster fork.
  Compare the read pattern of a heavy fork before and after, for example whether one file is still read in several slices.
