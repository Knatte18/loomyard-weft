# webster: keep earlier Masters' friction notes allowed after a resume

A resumed webster run could be blocked by its own run-exit audit.
The first Master's prompt named `webster-master.md` as its friction note, and Master wrote it.
After a resume, the new Master's prompt named `webster-master-2.md`,
and the audit, which allowed only the latest prompt's note, flagged the earlier write as `parent-write`.
Clearing it took a manual `webster accept-audit`.

The parent audit now lets Master write every friction note that any Master prompt of the run named,
both at `record-batch` and at the run-exit cross-check.

## Design decisions

- `State.MasterNotePaths` (`masterNotePaths` in `state.json`) lists the note paths in spawn order, each once.
  A spawn appends its path, and a spawn with friction notes off adds nothing and keeps the earlier entries.
- The old `masterNotePath` key is still read, so a run started by an older binary keeps its note when resumed.
  `LoadState` appends that path to the end of the list, if it is not already there,
  and clears the field, so the next save drops the key.
- `CheckParent` takes a slice of note paths in place of one path.
  The `parent-write` finding's detail text now names the notes of all the run's Master prompts.

## To check by hand

- The failing sequence (Master writes its note, the run is resumed, the run exits) has not been replayed on a live run.
  It is covered by `TestRun_DoneOutcome` rows with a scripted audit.
- Check the door call under `## Merge danger` against the `state.json` key rename from `masterNotePath`
  to `masterNotePaths`.
  It is called one-way because an older binary reading a newly saved `state.json` does not see the note paths;
  decide whether that case matters.

## Merge danger

**Door:** one-way
**Blast radius:** webster

A revert does not rewrite a `state.json` already saved with `masterNotePaths`.
An older binary ignores that key and drops the whole list when it saves the file.
Its audit of such a run can therefore flag even the current Master's note as `parent-write`.
Spawning a new Master records that Master's note under the old key,
and `webster accept-audit` clears a finding already raised.
