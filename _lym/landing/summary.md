# beam describe records the orch's own description, not an assembled one

`lym beam describe <slug>` used to assemble a beam's pull request description from the board entry, the beam's commits and the project record.
Its "Project decisions" section listed every entry `lym loom decision add` had added.
That list is a history of what was decided along the way, partly duplicated and partly superseded, and no reader of the pull request needs it.
The orch could change the description only through the entry's title and brief.

Now the orch writes the whole description, and `lym beam describe` validates and records it.

Closes #90

## What changes

- `lym beam describe <slug>` validates the orch's `summary.md` in the beam's `_lym/landing` directory and commits it.
  It stamps the description into the landing record.
  When the landing record names a pull request, it also runs the `lym beam pr update` sequence, so the pull request carries the recorded text.
  It no longer owns or rewrites the file.
- `lym beam describe <slug> --draft` writes a starting point: the entry's title as the heading, its brief, and a `## Changes` list of every member pull request.
  The orch edits that draft.
- The "Project decisions" section is gone from what lym produces.
- `contracts/orchrole/role-beams.md` tells the orch what to write:
  - the end result, what the beam ended up with, never the way there;
  - every member pull request, so a reader can follow each step to the result on its own;
  - never the decisions made along the way.
  
  The orch and the operator may change any part of the description at any time, then run `lym beam describe <slug>` again.
- The dobby notice about landings into a beam no longer tells the orch to re-run `lym beam describe`, since the orch's text is not regenerated.
  `docs/overview.md` and the package docs of `beamcli`, `landingshed` and `dobbyengine` follow.

## Design decisions

- In `internal/landingshed`, `WriteBeamDescription` is split into `DraftBeamDescription` and `RecordBeamDescription`.
  The draft is neither validated nor committed; recording does both.
- `--draft` refuses a `summary.md` that already stands, so it never overwrites the orch's text or a recorded description.
  The refusal names the file and the way forward: edit it and record it, or delete it for a fresh draft.
- Recording a missing file is refused with a way forward naming `--draft`.
  A validation finding is refused with the finding and the file to fix, and nothing is committed.
- The existing guards stay: the verb refuses a non-open beam, a pawl run not at Beam-Await, and a beam whose ready record stands.
  Together these keep anything at or after landing from rewriting the description.

## To check by hand

- Not run against a real hub: the `--draft` then record flow, and the pull request update when a pull request is recorded.
- If the landing record is unreadable after a successful record, the verb reports that the description is recorded and names `lym beam pr update <slug>` as the way forward.
  Check that message reads correctly.
