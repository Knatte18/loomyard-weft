# Fixer report — plan, round 1

## Changes

- **F1 (BLOCKING): fixed differently.**
  - Card 6 replaces `Profile.SeatTables` with `func (e *Engine) SeatTables(p *Profile, opts RunOpts) []seatengine.Table`.
    It returns `probeTable(p, opts)` and `p.fixTable(opts, nil, e.seatSkills(p))`.
    `probeTable` already passes `assistSeat`'s seat through `withAssistant` (`internal/burlerengine/review.go:154-158`).
  - The burler `Build` makes the engine with `burlerengine.New` and no shuttle, and passes `RunOpts` whose `Assist` names a stand-in model.
  - `Engine.assistSeat` joins the entry's `Holders`; `probeTable`, `assistSeat`, `withAssistant`, `seatSkills` and `New` join card 6's Uses.
  - Card 7's `unlistedLiterals` reports an unlisted `seatengine.Seat` literal, value or pointer, that sets `Stencil`, and gains the proposed test row.
    `internal/shedrecipe#multiLLMSeat` joins the real `listed` set, since its literal at `entries_multillm.go:164-166` sets `Stencil`.
  - Not made: the proposed `SeatTables(opts, skills, assist *seatengine.Seat)` with a stand-in assist seat passed in.
    `assistSeat` is a method of `*Engine` (`assist.go:115`), so `Build` cannot reach the real seat without an Engine.
    A seat hand-built in `shuttleparity` would be a seat literal the new scan rule reports, and it would not carry `assistReviewerStencil`, which is unexported.
    An Engine method reaching `probeTable` builds the assistant seat the way a probe of an assisted round does.
- **F2 (BLOCKING): fixed as proposed.** Card 6 drops `Seats`; `TestSeats_CodexManifestHoldsClaudes` calls `seatsOf(env, seatTables)` and fails on each builder error.
- **F3 (BLOCKING): fixed as proposed.** Card 2's `Manifest` gains `Verdict []ManifestItem`, and `WriteManifest` sets it from `m.Omitted()` before it writes.
  `TestManifest_Record` gains the round-trip row.
  Cards 3 and 4 assert the record's `Verdict` where they asserted the `Omitted` verdict.
- **F5 (MEDIUM): fixed differently.**
  - Card 6 adds `parityFixture.addSkills(t, names)`, which writes a SKILL.md per name under the fixture's codex.yaml `skill_roots` directory and the Claude config directory.
  - `TestSeats` and card 7's `TestProducers` call it with every skill name the built specs hold, before any `PrepareEach` runs.
  - Not made: having `newParityFixture` itself write the skills.
    The fixture seeds the stencils the builders render, so it exists before any spec does and cannot know the names.
- **F6 (MEDIUM): fixed as proposed.** `manifestEngine` moves from card 8's Create list to card 6's, and card 8's Uses gains it.
  Card 6's `TestCompare` gains the failing-Prepare and absent-record `PrepareEach` rows and the failing-builder `seatsOf` row.
  Card 8's `TestReport` drops the two `PrepareEach` rows and keeps its report-level rows.
- **F7 (MEDIUM): fixed as proposed.** Card 1 drops `TestStandingPassages`.
  The `TestPrepare_AppendsStandingPassages` bullet states that the Claude test's three rows keep pinning the zero `Standing`.
  The ImpactSummary's test count is gone.
- **F8 (LOW): fixed as proposed.** Card 2 now says an attempt that fails after its run record is saved, such as one whose startup the start retry relaunches, keeps its copy under its own run id; one that fails before then leaves none.
- **F9 (LOW): fixed differently, under the F4 ruling.** Card 7 names `internal/loomcli#driverSpec` beside `internal/websterengine#Run` as compared like every other producer.
  The ruling removed "logged and not compared", which the proposed fix assumed.
- **F10 (LOW): fixed as proposed.** Card 2 gives the unreadable-record warning its own way forward, naming the run directory's `manifest.json` and the engine's `WriteManifest`.
  The directory way forward stays on the failed copy write.
- **F11 (LOW): fixed as proposed, corrected in the self-check.** Card 3 adds `internal/dotgit#FindRoot` to its Uses.
  It derives the root as `readCheckout` does: `FindRoot` on the anchor, then `filepath.EvalSymlinks` on the root.
- **F12 (NIT): fixed as proposed.** Card 7 names `shedarbiter.Prompt` and `gatefixer.RenderPrompt`.

## Self-check

- fix-consistency, LOW, overview intro and card 7's index line not naming the Seat-literal scan: fixed as proposed.
- fix-claims, MEDIUM, card 3 had `FindRoot` run on the resolved anchor, unlike `readCheckout` (`home.go:74-78`): fixed as proposed.
  `FindRoot` now runs on the anchor and `EvalSymlinks` on the root.
- fix-claims, MEDIUM, card 1 tied the Warn log to all three rows, though only `reader_error` asserts it: fixed as proposed.
- fix-proposal, BLOCKING, the F4 ruling leaves every `ForkSubagents` seat and producer missing `guard webster-fork-deny` on Codex.
  I flagged it to the parent, and the parent's fork-deny ruling settled it; see Rulings.
- fix-proposal, MEDIUM, card 3 left no fallback when `FindRoot` finds no repository, and its fixture anchor has no `.git` (`prepare_test.go:72-82`): fixed as proposed.
  With no repository, card 3 lists only the working directory's CLAUDE.md under key `CLAUDE.md` and the launch goes on.
  The test fixture writes a `.git` directory at the anchor.
- fix-proposal, LOW, the stored `Verdict` lists every unrun plugin context in every parity and verb record: fixed with my own wording.
  Card 8 states that the verb prints `Missing` as its verdict, and that only a real launch's record verdict says whether a context was delivered.
- fix-affected-set, MEDIUM, the fanned assistant stencil (`assist.go:142-144`) was outside the catalog: fixed with my own change, merged with the next finding.
- fix-affected-set, MEDIUM, the fanned burler review and fix seats the ruling names were not built (`review.go:87`, `fix.go:97`): fixed with my own change.
  The burler `Build` now calls `SeatTables` with a solo `Profile` and with a fanned one.
  The fanned one's `ClusterFan`, `FixFan` and `opts.AssistFan` name fans from `lenses.yaml`, loaded with `lenses.Load`.
  The two profiles carry distinct stand-in `RolePrefix` values, so no two catalog seats share a name.

## Disputed

None.

## Deferred

None.

## Rulings

- **F4: compare every seat and producer, a seat Codex cannot start included, and drop the Unmet skip.**
  This came as a parent decision recorded through `lym loom decision add`.
  - Overview: the decision is renamed "every seat is compared, a seat Codex cannot start included".
    It cites the record's entry and keeps `lym shuttle manifest` naming unmet requirements through `Unmet`.
  - Card 6: an Intent sentence says neither the tests nor `PrepareEach` skip a spec `Unmet` reports.
    `TestSeats` compares every `Seat`, and its skip sentence is gone.
  - Card 7: `TestProducers` compares every producer, and its Uses drops `Unmet`.
    `Unmet` keeps its production caller in card 8's `report`.
  - The ruling is carried out.
    I told the parent once that it left `guard webster-fork-deny` missing on Codex for every fork seat, since Codex has no fork deny and `steerWebsterForkDeny` is Claude's alone (`claudeengine/settings.go:25-29`).
- **Self-check fix-proposal BLOCKING: `webster-fork-deny` is compared only on an engine that meets `RequirementForks`; a test pins the tripwire; no card builds a Codex fork deny.**
  This came as a parent decision recorded through `lym loom decision add`, and it settles the flag above.
  - Card 2 creates `shuttleengine.WebsterForkDenyKey`, and card 3 lists the guard item under it.
  - Card 6's `Compare` becomes `Compare(engines, manifests)`.
    It drops that item while `Unmet` reports `RequirementForks` for the set's `codex` engine.
    `TestCompare` gains the tripwire rows: a `manifestEngine` meeting no requirement gives no finding, and one meeting `RequirementForks` gives the item.
  - The overview decision states the gate and cites the entry.
  - Every other item of a fork seat stays compared.
