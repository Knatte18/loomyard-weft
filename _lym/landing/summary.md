# Report: revise the remaining adopt-build members against what landed

A read-only study of the six members of the beam `adopt-build` that have not run, read against the run FSM and the orch's own-session resume that landed in it.
The beam's bodies and records were written before those two landed and before the operator dropped the orch transcript mirror (F12) and kept the orch up during an adopt hold (F14).
The study changes no production code, board entry or decision record; the orch applies its result once the operator approves it.

## What changes

- `docs/research/loom/adopt-build-revision-2026-11-16/report.md` holds one section per remaining member:
  what changes and why, the revised body in full, the order, dependency and recipe changes, and the open questions for the operator.
- Beside it, `record-*.md` hold the revised decision records of `push-before-act-takeover`, `bolt-holds-catch-up` and `adopt-foreground-leftovers`, and a new record for `leftover-pairs-teardown`.
- `docs/research/README.md` lists the report.

## Findings that shape the revisions

- The landed run FSM differs from the Sketch where later parts depend on it.
  One reader, `pairteardown.ReadRun`, fills the act rule's inputs, and `Held`, `PushOwedOffline` and `CodeTipMissing` are declared and never set.
  The act rule has `Judge` and `Aside`, which the Sketch lacks.
  Pause and cancel intents push through a detached push, with no owed-push record.
- The landed subject-process end, `pairteardown.EndSubjectProcesses`, has no dobby caller yet.
  `tendercli.PauseOwnedElsewhere` and the dobby's `PauseOwned` seam still pause under the `owner` caller, and `bolt-holds-catch-up` has to replace them.
- The upgrade's first read of standing pause entries is in two members.
  It belongs in `bolt-holds-catch-up`, where the hold setter needs a production caller under `PATTERN-test-only-code`.
- `prime-is-root-beam` changes only what the board prints.
  Moving the code's empty parent for the prime to a name would reach every caller of `hubgeom.ParentOf`.
- `hub-stop-button` must not call `lym tender pause <slug>`, which now writes and pushes a tracked intent.
  It reuses hibernate's pause-and-confirm sequence and ends the orch's registered Claude process, which outlives its strand.
- `machine-restart-shortcut` must not start a hub through `lym s`, whose `lym adopt --here` takes over a hub another machine owns.
  It builds on `lym beam start --all`.

## Design decisions

- The report recommends splitting the leftover-pair teardown (D11) out of `adopt-foreground-leftovers` into `leftover-pairs-teardown`.
  It shares no code or decision with the foreground adopt.
  The report gives the merged form if the operator declines.
- The revised records fold the Added-after-Discussion entries into their decisions and carry none.
- Dependencies, recipes and the linear order of the chain stay.
  The report marks which dependencies are ordering only.
- Five choices are left to the operator, each with a recommendation: the split, the printed name of the prime beam, the stop's pause entry, which hubs the restart starts, and the order of the darn members.

## To check by hand

- The report's claims marked *not read*: the cancel path's push, the PR record's and fixer counts' pushes, what `lym tender resume` reconciles, and the `loop-exited` halt.
- The changed-line counts in the revised package lists are the Sketch's estimates adjusted by reading, not measurements.
