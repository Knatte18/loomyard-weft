# Webster's parent audit allows every friction note the run's Masters got

A resumed Webster run spawns a fresh Master, and because the earlier Master's friction note already exists, the new prompt names the next free path, such as `webster-master-2.md` beside `webster-master.md`.
The run state kept only the latest prompt's note path, so the run-exit audit and record-batch's parent audit treated the earlier Master's note as an unauthorised parent write.
The run then stopped on a correctness finding over a file Master had been told to write, and someone had to accept the finding by hand.

## Changes

- `State.MasterNotePath` (`masterNotePath`) becomes `State.MasterNotePaths` (`masterNotePaths`): every friction note path a Master prompt of the run named, in spawn order, each once.
  `Run` appends the new prompt's path at each Master spawn and skips a path already listed, which happens when an earlier Master never wrote its note.
- `CheckParent` takes the list and allows a parent write to any listed note, each compared in its told and link-resolved spellings.
  An empty list allows no note, as an empty path did before.
- Both callers, the run-exit audit in `runlevel.go` and record-batch's incremental audit, pass the list.
- The package doc and `audit.go`'s file header now describe the allowance as covering every note the run's Master prompts named.

## Design decisions

- A `--fresh` run starts a new state, so its list starts empty; notes from an archived run are not carried over.
- The old `masterNotePath` key is not read back.
  A run whose state was written before this change and that resumes after it starts its list with the new prompt's path alone.
  Its earlier note can still raise the old finding once, and the existing accept route clears it.

## Tests

- `TestRun_DoneOutcome` gains a row in which an earlier Master's note exists and is recorded in the state.
  The resumed Master's prompt names `webster-master-2.md`, and the audit shows writes to both notes.
  The run ends done with no parent-write finding, and the state lists both paths in order.
- The record-batch row on Master's friction note now records two note paths and writes both, with no finding.
- `TestCheckParent` gains a row allowing a write to an earlier prompt's note and a row refusing a note write when no note was named.
- The persisted-identities row in the run tests now checks the list holds exactly the prompt's note.

## Merge danger

**Door:** two-way
**Blast radius:** webster
