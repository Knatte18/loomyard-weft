# Agents read a file whole or all its spans in one message, and not again

Webster forks read one file in three to five slices over as many turns and re-read files they already held, each turn paid at full context.
This change asks for a file to be read whole, or every span needed from it, in one message, and for a file read once to be kept in mind.

## What changes

- `contracts/stencils/webster/webster-body-implementer.md`: step 2 tells the implementer to read every file a card needs in one message before the card's first edit, whole when it needs most of the file and otherwise every span it needs, never one file in slices over several turns.
  A file read in the batch is read again only where a change the agent did not make by hand, such as a rename tool's run, touched it, never to check its own edit.
  The `lym quarry resolve` sentence no longer forbids whole-file reads.
- `internal/shuttleengine/claudeengine/settings.go`: the batching sentence every Claude session gets in its appended system prompt now says to read a file whole, or every span needed from it, in one message, and to keep a file already read in mind rather than reading it again.
  The package doc in `doc.go` names the addition.

## Design decisions

- The sentence stays in `claudeengine` where `main` holds it.
  The `batching-in-claude-local` run moves it into heddle's `shipped.json`; whichever run lands second carries this wording into the moved copy.
