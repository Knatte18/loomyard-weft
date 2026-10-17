# Support log — creel CLI output fixes

## Interview

### Turn 1 — exploration and auto-picks

Autonomous run: no operator answers, every pick below is the writer's own.

- Read the board entry (#633) and the board; no other entry plans changes to `internal/creelcli` output or `Store.Log`.
  `creel-notices` (stage 2) and `creel-agents` (stage 3) are later stages that move callers onto creel; neither touches these paths.
  `stencilstore-dev-warning-noise` touches stderr noise on creel verbs from stencilstore, a different source.
- Read `runVerb`, `writeRefusal`, `writeOk` and every verb body in `internal/creelcli/cli.go`, `output.Ok` in `internal/output`, and `Store.Log` with `LogEntry` in `internal/creelengine`.
- Confirmed each cause the entry names: `output.Ok` discards the write error; `writeRefusal` prepends `creel: ` unconditionally while engine errors already open with it; `Store.Log` compares `ID` only.
- Found that creelcli's own refusals mostly lack the prefix (`creel needs a hub ...`, `post takes ...`, `log: --since ...`), so stripping the prefix from the fallback would leave those unprefixed; a conditional prefix fixes both.
- Found that mail and injects share one id counter, so matching `Unread` cannot alias an inject id.
- Found exact-list assertions on `LogFilter{ID: ...}` in `deliver_test.go`, recorded as an open risk for the plan.
- Left out `docs/research/creel-audit-2026-10-16/report.md`, a dated audit report that mentions `creel log`.

### Turn 2 — lens review

- goal-scope, premises, scenarios, leanness (merged): the wait timeout stores an id the refusal never names — fixed: read `Client.Call`, which answers `ErrWaitTimeout` with an empty response; recorded as out of scope with the reason and as an open risk, and D2's wrong reason replaced.
- goal-scope: D2's widening past the entry is inconsistent — fixed by the same change: D2 now covers every result that holds the id, and the timeout exclusion has its reason.
- goal-scope: `creel: creel needs a hub` reads doubled — rejected: it is one prefix plus the message's own wording; the entry's examples are engine errors, and rewording creelcli errors is outside the entry.
- goal-scope: criteria name only four verbs — fixed: D1 names every `runVerb` verb.
- goal-scope: `--wait-reply` loss glossed — fixed: Open risks states the reply is marked delivered and lives only in the log.
- board-fit: sandbox suite's `log --id` line goes stale — fixed: added to Scope.
- board-fit: no sandbox `/dev/full` cases — rejected: the creelcli failing-writer subtests pin each case, and `PATTERN-sandbox-coverage` is met.
- board-fit: `stencilstore-dev-warning-noise` stderr overlap — fixed: Open risks, and criteria name the command's own stderr writer.
- premises: the EXTERNAL inbox refusal is an engine error — fixed: criteria now name a standalone directory and a wrong argument count as the creelcli examples.
- premises, scenarios: `--id` with `--from` drops reminders — fixed: stated in D4.
- pattern-fit: test tier for D4 — fixed: Constraints pins D4 untagged in creelengine and D1–D3 in the integration-tier creelcli suite.
- scenarios: inbox from a terminal is a refusal — fixed: criteria say "from a strand".
- scenarios: expired and returned cases have no scenario — fixed: criteria list exits 3 and 5 with the id.
- scenarios: both streams refusing — fixed: D1's bound states the exit stays 1.
- scenarios: `<done>` with D3 — fixed: D3 states the line shape.
- leanness: exit-1 reason stated twice, D2 rationale duplicated, Notes for the plan writer repeats `relevant.md`, Scope tests bullet, docs/research exclusion, comment-line-breaks constraint, open-risk justification clause — fixed: each cut or moved here.

## Rejected alternatives

- **Make `output.Ok` return the write error, or check it there**: it is shared by many modules whose callers ignore its return; changing it widens the task past creel for no asked-for gain.
- **Strip `creel: ` from engine errors**: engine errors reach other callers (Go clients, logs) that rely on the prefix.
- **Never prepend in the fallback**: creelcli's own refusals would print with no `creel: ` opening, against the doc line.
- **A new exit code for a lost answer**: the entry asks for the refusal's behaviour; exit 1 is what a caller already handles.
- **Name the id by reading `fields["id"]` in `runVerb`**: implicit; a later verb adding an `id` field would change its stderr line by accident. An explicit result field states intent.
- **Carry the acknowledged id out of `Client.Call` on a wait timeout**: a client API change for a gap that exists on a writable stdout too, outside this entry's three bugs.
- **Add an `ids` field to the `reminded` record so `ID` matching works**: changes the journal shape for a filter fix.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): exit code for a success envelope that cannot be written? — 1, as a refusal.
- Q2 (auto-pick): how the stderr line names what was stored? — an explicit result field set by post and inject (D2).
- Q3 (auto-pick): does the fallback of non-zero post and inject results name the id too? — yes, same field (D2).
- Q4 (auto-pick): fix the doubled prefix where? — conditional prefix in `writeRefusal` (D3).
- Q5 (auto-pick): `--id` matches on `Unread` for every event or only `reminded`? — every event, event-agnostic (D4).
- Q6 (auto-pick): the hidden daemon verb's `output.Ok`? — out of scope; its stdout is the spawn's.
- Q7 (auto-pick): name the id of a waiting post or inject that times out? — out of scope: the client drops the id; recorded as an open risk.
