# Support log — run-time-by-role

## Interview

Autonomous run: no operator answered; every decision below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the related entries `gate-checklist-pane` (an idea for a per-gate timing pane, not overlapping) and `convergence-plan` (intervention logging, not overlapping).
2. Read `tools/tokencount`: per-run role tallies from transcripts, an active window per run, the orch charge, and calibration reads from the weft through `gitrepo`.
3. Read an archived run (`archive/plan-gaps/...` in the prime's weft): `seed.json` carries `started_at`; every `status.json` history entry carries its finish time `at`, folded entries `last_at`; no start time; `PR-Gate` appears as `awaiting` then `done`.
4. Read that run's transcripts: the driver launches `lyx shed step --until-stop` as a background Bash call and ends its turn, resuming on a task notification, so turn boundaries split work from waits; sampled `stop_reason` values are only `end_turn` and `tool_use`, none null.
5. Checked PATTERN: `PATTERN-shed-run-directory` and `PATTERN-lyxdirs-single-declarer` force paths and seed decoding through `shedrun`; `PATTERN-fabric-git` exempts reads.
6. Settled the design: turn-based working time, union per role, idle from the archived span, a Producers table from history gaps, no shedengine change.

## Rejected alternatives

- **Working time as first-to-last line per session.** Counts a writer's wait for parent review and a driver's whole run as work; the driver would read near 100%.
- **Working time by an idle-gap threshold.** A threshold misclassifies long thinking or tool calls and short waits; turn boundaries are recorded and exact.
- **Summing a role's sessions instead of their union.** Parallel forks and reviewers would exceed the span and break the share.
- **Partitioning each instant to one role (parent wins).** Shares would sum to 100%, but `+sub` rows would read near zero because Merriam is in its turn while forks run, hiding fork time.
- **Adding a `started_at` to shed history entries.** The finish times already tile the run; a start only splits a gap into before-start and in-row, which the board does not ask for, and no archive has it.
- **Splitting idle into Go work and waiting.** The records cannot tell them apart without new instrumentation the board does not ask for.
- **Reading live status for in-flight runs.** The tool counts finished runs; an in-flight run's later rows are missing anyway.
- **A separate time table instead of new columns.** The board asks for time beside tokens.
- **Renaming the output file.** No request for it, and it would break the operator's path.

## Review rounds

_No rounds yet._

## Question ledger

- Q: What counts as a session working? — Auto-pick: in a turn, by `stop_reason` boundaries. Resolved.
- Q: Do `+sub` rows get time? — Auto-pick: yes, as their own union, as for tokens. Resolved.
- Q: Where does the run's span come from? — Auto-pick: seed `started_at` to last history end; transcript fallbacks. Resolved.
- Q: One idle line, or idle per row? — Auto-pick: one `idle` row per role table, and an idle column in the Producers table. Resolved.
- Q: Add start timestamps to shed history? — Auto-pick: no; finish times exist, so the board's condition does not hold. Resolved.
- Q: Is a per-producer table in scope? — Auto-pick: yes, it is how "time each producer takes" reads in shed terms, from data the board says to read. Resolved.
- Q: Which archive tag when a slug has several? — Auto-pick: newest tagged commit by committer date. Resolved.
- Q: Weft default when slugs are named? — Auto-pick: the default applies whenever `-weft` is empty. Resolved.
- Q: Orch time? — Auto-pick: blank; orch spans runs. Resolved.
- Q: Report path and title? — Auto-pick: path unchanged, title "Token usage and time by role". Resolved.
- Note: `tools/tokencount` sits near its `test-budget.yaml` row; a card whose new top-level tests push past it lists that file under `Edit:`.
