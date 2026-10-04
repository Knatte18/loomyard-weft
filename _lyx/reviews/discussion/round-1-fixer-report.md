## Fixed

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D10 now refuses a pull request number on `import` and `close`, an unrecorded closed issue, and a recorded issue on `close`; matching acceptance criterion added.
- F2: D10 Bound states the retry names the recording entry's slug, is reachable only by explicit `import`, and skips an existing comment.
- F3: D10 states `into` accepts any kind and status under D3 validation, and refuses `title`, `brief` and `labels` beside it; acceptance criterion added.
- F4: D12 states a hub with area labels needs one `board.yaml` edit, status writes included; rationale reworded from "migrates" to "converts".
- F5: D6 refuses a filter label only when it is in neither config list and no entry carries it; a path-only `Board` does not validate; acceptance criterion added.
- F6: D12 drops repeated labels (first kept) and names legacy and configured type names.
- F7: D8 Bound states what the flag reads and refuses stdin when the payload is also on stdin; acceptance criterion added.
- F8: D9 states `issues` replaces the list and refuses duplicates; acceptance criterion added.

## Deferred

Nothing deferred.
