---
round: 1
ledger:
  - key: once-marker-gate-definition-and-lifetime
    rounds: [1]
    status: resolved
  - key: shared-info-exclude-write-safety
    rounds: [1]
    status: resolved
  - key: tracked-tasks-json-overwrite
    rounds: [1]
    status: resolved
  - key: lost-scratch-reopen-behaviour
    rounds: [1]
    status: resolved
  - key: attach-targets-child-session-premise
    rounds: [1]
    status: resolved
  - key: folder-decision-missing-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: spawn-confirmed-marker-misnamed
    rounds: [1]
    status: resolved
  - key: closure-decision-signature-and-deliberation
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings against the batten-open-ide discussion pair; the fixer addressed all eight before judging, and each was verified in the current files.

The substantive ones were:
the once-per-run gate was ambiguous between "no marker present" and "first spawn", and had no lifetime across runs of the same slug;
the `info/exclude` write ignored that the file is shared repo-wide and must go through `fabricengine`'s lock with a subpath-matching pattern;
and overwriting `tasks.json` contradicted the no-tracked-file rule for repos that commit `.vscode/tasks.json`.
The rest tightened the record: lost-scratch behaviour, the attach-reaches-the-child's-session requirement, a folder criterion, a marker name, and a relocation of deliberation out of a Decision.

No finding remains open.
