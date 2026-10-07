# Round 3 fixer report

All six findings fixed in `_lyx/discussion/decision-record.md`.
`lyx loom validate-discussion` passes.
`support-log.md` is unchanged.

## Changed

- F1 (fixer done before the marker): D6 now treats a fixer run that reaches done before the marker as a skipped handoff.
  Go stops the reviewer and removes its strand, and the round returns an error naming the skipped handoff, with Verdict and Findings empty.
  D1 points at that rule, and the acceptance criterion on failing halves carries the case.
- F2 (producer's resume capabilities): D6 says the probe result carries each live half's strand guid and that `BurlerProducer` is told the same strand remover as the engine.
  D5 names the steps from the reviewer's done outcome to the marker write (fork audit, strict parse, marker) as one function, called by both the engine's reviewer goroutine and the resume branch.
- F3 (review file bound): D7 has Go keep the parsed review bytes and compare the review file with them once the fixer's run reached done.
  A difference returns an error naming the file, with Verdict and Findings empty.
  A new acceptance criterion covers it, and the Open risks line says the fixer's other write limits are prompt-enforced.
- F4 (classification wording): D6 defines live, done and gone by the probe result and the output file, and calls a marker found in the both-live branch stale.
- F5 (D5 bound): D5 now says only lyx code writes the marker and that the fixer's prompt write surface is what keeps it from creating the marker, a prompt-enforced rule named in the Open risks.
- F6 (standalone marker location): D5 says `.lyx` in the told state directory, matching the Constraints.

## Deferred

Nothing deferred.
