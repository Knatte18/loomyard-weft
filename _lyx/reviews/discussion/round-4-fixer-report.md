# Round 4 fixer report — discussion

All eight findings were validated against the code and fixed.

## Fixed

- **F1 (BLOCKING)** — Validated: under `strace`, `cat >> f && printf '\n' >> f` appends a 20000-byte payload in one `write`, and the `$(date)` stamp line leaves in one short `write`.
  D1 no longer rewrites the payload append: every hook appends a short stamp line (`{"lyx_stamp":"<hook>","lyx_at":"<time>"}`) by one `printf`, then its payload by today's unchanged `cat` command, joined by `;`.
  The Stop and ask payload lines are byte-identical to today's, so the Bound now holds as stated.
  The remaining exposure, more appenders hitting today's payload-to-newline gap, is stated in D1 and added to Open risks with its consequence.
  The pairing rule (nearest preceding unclaimed stamp naming the payload's `hook_event_name`) and the same-name swap are stated.
  The new hooks end in `; true` so they still exit 0.
  The acceptance row is rewritten; the inline-stamp form is added to Rejected alternatives with the measurement.
- **F2 (MEDIUM)** — Validated: the 2.1.294 binary's `SessionEnd` reasons are `clear`, `resume`, `logout`, `prompt_input_exit`, `other`.
  `dead` now covers only process-ending reasons; `clear`/`resume` read `unknown` with the reason until the next signal; precedence rule 1, the `since` list and the acceptance rows say so.
- **F3 (MEDIUM)** — Validated that the session id has no source after `/clear`, and found the record's own premise false: the orch has a run record (`Runner.ReadEvents` → `FindRun`, `internal/shuttleengine/session.go:70`), interactive, with a never-written sentinel as its one output file (`internal/orchcli/start.go:61-70`).
  D3's facts list now reads the orch from its own record.
  Its turn ends read interactive `asking`, not `idle-done`; the acceptance row is corrected.
  The liveness session id is now the newest hook line's `session_id`, else `run.json`'s, in D3 and D4, with an acceptance row.
  The registry's behavior across `/clear` is added to Open risks.
- **F4 (MEDIUM)** — `interrupt` and transcript-marked `api-error` moved into a per-reader group (transcript marker) in D3.
  Both marker reads apply only to the newest turn end or turn start of a fold, stated in the facts list.
  An acceptance row covers an older turn end followed by a newer one.
- **F5 (LOW)** — Observed every entry in `~/.claude/sessions` naming a live pid.
  D4 now states the likely premise, and that `died` against `unknown` is logged as a disagreement on purpose.
  Open risks lists it as unverified.
- **F6 (LOW)** — D3 now states that the wait loop keeps its reducer result between polls and re-judges only the newest `no-output`/interactive-`asking` turn end against a later poll's stat, leaving earlier entries as logged; the acceptance row says so.
- **F7 (LOW)** — D3, D6 and the support log's rejected alternative now all use "the newest history entry before it that is not a call-time state", with the `since` D3 gives it.
- **F8 (NIT)** — Resolved by F1's design: the payload line is unchanged, so the events are equal, `Raw` included.
  The acceptance row names a 20000-byte payload, past the 4096-byte `printf` chunk.
  Constraints name the `integration` tier for the real-shell round trip.

## Disputed

None.

## Deferred

None.
