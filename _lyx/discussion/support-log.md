# Support log — engine-usage-readers

## Interview

Autonomous mode: no operator turns; each topic below was settled by auto-pick after reading the code.

1. Scope — what "deletes its own parser" covers.
   tokencount's parser reads turn time, custom titles, fork card prompts, begin-batch results and the Merriam launch-line Read, not only usage.
   Picked: the neutral record carries every fact those reads use (D2), so the whole parser goes.
2. How tokencount finds a run's engine.
   Shuttle records the engine in `RunState.Engine` under `.lyx`, which a torn-down pair loses.
   Picked: locate by worktree path across every registered engine (D1).
3. Where rules split between engine and tool.
   Picked: provider format rules in the engine, lyx prompt-format rules in tokencount (D2); the launch line is rendered by `claudeengine/command.go`, so its recognition is the engine's.
4. Prices.
   Picked: a pricing capability per engine, tally and rendering stay in tokencount (D4).
5. Building engines without config, and the `-projects` flag.
   Picked: an offline engine set from `shuttleengines`, injected into tokencount; `-projects` deleted (D6).
6. Failure handling.
   Picked: tokencount stays loud; the engine's `SessionUsage` stays degrading (D7).
7. Calibration instruction files.
   Picked: the Webster session's engine's instruction-file list at the base commit (D8).
8. Lens review (one fork per lens, all six returned):
   - premises: duplicate figure is in the "All runs" total line, not the header — fixed (D9 names it).
   - premises / scenarios: id-less messages keyed differently by `SessionUsage` and tokencount — fixed (D2 message key, D5, acceptance).
   - premises: "imports only shuttleengine and shuttleengines" overstated — fixed (of the engine packages).
   - premises: D1 run-record premise confirmed — no change.
   - goal-scope / premises / scenarios: duplicate-count choice left to the plan — fixed (D9 decides it; Q7 closed).
   - goal-scope: Out contradicted D7 and the duplicate change — fixed (Out names the one exception).
   - goal-scope: D7 header line unlicensed — fixed (named as an addition in Scope); kept, since a second engine would otherwise undercount silently.
   - goal-scope: neutral shape fitted to Claude — fixed (Open risks).
   - goal-scope: relation to `UsageReader` unstated — fixed (D5).
   - board-fit: `round-N-usage.yaml` overlap with `engine-model-choice` — fixed (Open risks).
   - board-fit: `codex-engine` must implement the capabilities — fixed (Open risks); a board edit is the orch's.
   - board-fit: `report-timing` has no board entry — fixed (Out).
   - board-fit: `session-sync` path mismatch — rejected: a Someday note, no design impact.
   - leanness: contrast clauses in D1, D6, D7 rationale — fixed (trimmed; alternatives stay in Rejected alternatives).
   - leanness: auto-mode bullets repeating D1 and pointing at the support log — fixed (cut).
   - leanness: Constraints' unchanged-output bullet repeats Out — fixed (merged into Out).
   - leanness: Notes bullet repeating D6 — fixed (cut).
   - leanness: gate-pass criterion generic — fixed (folded the seam scan into the first criterion).
   - leanness: docs bullet restates CLAUDE.md — rejected: it names the specific `doc.go` files.
   - scenarios: D6 unbounded zero-config engines — fixed (read capabilities only, no `Engine`).
   - scenarios: per-engine empty state and locator errors — fixed (D7, acceptance).
   - scenarios: cross-engine dedupe — fixed (keyed by engine and message key).
   - scenarios: cutoff cases unpinned — fixed (acceptance).
   - scenarios: session vanishing between locate and read — fixed (absent, D7).
   - pattern-fit: HOME-based tests cannot run parallel — fixed (root as parameter, D6).
   - pattern-fit: told-geometry wording overstated — fixed (Constraints).
   - pattern-fit: fake ownership — fixed (test-local fake, D6 and Constraints).
   - pattern-fit: refusal-way-forward does not apply — no change.

## Rejected alternatives

- Read the run's engine from shuttle run records: they live under `.lyx` and vanish with the pair, so finished runs, the default selection, would have no engine.
- A usage-only neutral record, keeping calibration and turn time on a Claude parser in tokencount: leaves the parser the board entry asks to delete, and keeps a provider specific outside the engine.
- Source fork cards, the base SHA and the Merriam start from webster run records instead of transcripts: changes what the report measures, which this task does not license.
- Keep `-projects` as a generic root override: the root is a Claude layout fact; a neutral flag naming one engine's directory would leak it back.
- Build the engine set through `shuttleengines.New` with the hub's `shuttle.yaml`: needs a default engine and a valid config the read does not use, and fails the report on unrelated config errors.
- Let tokencount degrade to unknown like `SessionUsage`: a silent undercount in an operator cost report; `SessionUsage` degrades because shuttle must not fail a run on a report-only read, which tokencount does not share.
- Reword the repeated-lines figure to count cross-transcript repeats only: changes a figure the report shows today.
- Replace `UsageReader` with the new reader: it serves a run's finalize with different failure rules.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Does the neutral record cover calibration's reads? Yes, D2. Resolved.
- Q2 (auto-pick): How is a finished run's engine known? By locating sessions per worktree across engines, D1. Resolved.
- Q3 (auto-pick): Where does the price table go? `claudeengine`, behind a pricing capability, D4. Resolved.
- Q4 (auto-pick): Keep `-projects`? No, deleted, D6. Resolved.
- Q5 (auto-pick): Fail or degrade on an unreadable session? Fail, D7. Resolved.
- Q6 (auto-pick): Calibration's instruction files? The Webster session engine's list at the base commit, D8. Resolved.
- Q7 (auto-pick): the repeated-lines figure once the engine collapses streamed lines? The reader reports its collapsed count and the figure keeps its meaning, D9. Resolved.
- Q8 (auto-pick): what the offline set exposes? Read capabilities only, D6. Resolved.
