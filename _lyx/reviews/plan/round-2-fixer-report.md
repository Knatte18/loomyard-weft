# Round 2 fixer report

All four findings fixed; nothing deferred.
The `approved:` key in `00-overview.md` is untouched.

## What changed

- F1 (`09-verify-timeout-findings.md`): added `contracts/specs/webster-spec.md` and `docs/overview.md` to the Edit list.
  The Intent gains the sentence that both gain the exception to "a failing run is rerun once": a run that exceeds the verify timeout is not rerun and fails the gate at once with the timeout, log path and log tail as findings.
  The ImpactSummary names the two added docs.
- F2 (`10-verify-wait-note.md`, `00-overview.md`): the status note is now `<producer>: verify running <elapsed> (attempt <n>; <command>)`, with no site clause, and `Marker.Site` stays in the marker.
  The test sentence drops "site shown and hidden".
  The first Shared Decisions bullet now reads `verify` for the status note and `verify <producer>` on screen.
- F3 (`16-batten-respawn-scoped.md`): added `docs/overview.md` and `internal/battenshed/doc.go` to the Edit list.
  A new Intent paragraph rewrites the overview's two spawn-confirmation clauses and has `doc.go` record the process-scoped rule and the adoption cases.
  The ImpactSummary names the two docs.
- F4 (`05-reed-resume-done-when.md`): the drop rule no longer carries a not-hidden condition, so a finished hidden strand is dropped like any other non-live strand.
  The table sentence now pins a hidden strand with a fully present list as dropped, and a hidden strand with no list or a partly missing list as neither launched nor dropped.

## Deferred

None.
