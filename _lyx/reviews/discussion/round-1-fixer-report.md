# Fixer report — discussion round 1

All seven findings fixed; `lyx loom validate-discussion` exits 0 on the result.

## Changed

- **F1 (MEDIUM):** the Command-source Decision now says `Deps` carries a closure that `internal/loomcli` fills to parse the plan when the gate runs, why a bootstrap-time value would silently skip, and that a plan read or parse error is a returned error, never a skip.
  "No command means no gate" now says only a parsed plan with no `## verify:` section yields an empty command.
  New acceptance criterion for the read or parse error.
- **F2 (MEDIUM):** an empty command skips the gate whether or not a marker exists, leaves the marker alone, and names it in the warning;
  a marker write or output-file create failure is a returned error, like a `CommitStatus` failure.
  New acceptance criteria: empty command with a marker present, write failure, and `Finalize` with a no-op merge-in and a marker present;
  the spawn-failure criterion now says the marker remains.
- **F3 (LOW):** a new Decision, "One shared runner, webster's behaviour unchanged", says webster passes a background context and a discarding writer and that the spawn and teardown log lines stay;
  the webster acceptance criterion names the integration-failure bisect and the log assertions.
- **F4 (LOW):** the Execution Decision now states the kill-the-shell-plus-bounded-wait form `internal/treadleengine` uses, and Open risks gains "Orphaned children of a cancelled verify".
- **F5 (LOW):** the Goal's first line is scoped to what the plan's `## verify:` covers.
- **F6 (LOW):** the Goal's incident detail is cut to one sentence (a merge can compile and still break tests), and the command-by-command detail plus the coverage finding moved to support-log Interview turn 1;
  Auto-mode assumptions keep only the Finalize departure plus a pointer to the Question ledger;
  the "Output file size" risk is dropped.
- **F7 (NIT):** the Trigger Decision now says a stuck resolver outcome returns before the gate with no verify and no marker, with a matching acceptance criterion; the Notes bullet stays as orientation.
- `support-log.md`'s Question ledger gains Q11–Q15 for the self-picks these fixes made.

## Deferred with reason

None.
