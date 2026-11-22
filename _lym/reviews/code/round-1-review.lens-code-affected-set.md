```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines they touch.

**Per change, the set found and whether it is covered**

1. **`gateslot.HubSetsGate` removed, with its test.**
   Callers: only `hardenLiveKey`, now gone, and `git grep` finds no other.
   The `gateslot/doc.go` line is removed.
   The `gateslot/template.yaml` comment is rewritten.
   `PATTERN-test-speed`'s gate-key table carries no hub-writes claim.
   One fixture is stale: `internal/preflightshed/preflight_test.go:172` still uses the retired refusal text (finding F4).
2. **`LiveKey.Set` removed; `hardenLiveKey` takes no dirs and returns no error.**
   Callers: `hardenPreflight`, `TestHardenLiveKey` and `TestCheckCampaign`, all updated. Covered.
3. **`CheckCampaign` accepts an empty package list and template live keys.**
   The function doc, `hardenshed/doc.go` and `template.yaml`'s `packages` comment are updated.
   The refusal list in the doc is updated. Covered.
   Checked against `PATTERN-config-layers` (a meaning change ships a reinterpretation): `packages: []` used to halt Preflight, so no hub ever ran with it.
   No working value changes behaviour, and I judge it no violation.
4. **`copiesRun` now gates on packages, and the live mark ignores "started none" with no packages.**
   `live.go`'s mark constants and the `NewLiveGate` doc are updated, as are `hardenshed/doc.go` and `TestLiveGate`.
   `gatecli`'s `--copies` refusal of an empty target list (`run.go:400`) matches the new reason. Covered.
5. **`Campaign.Entry` and `Campaign.SeedBrief` added.**
   Wired in `hardenwiring.go` `hardenCampaign` and in `entries_bouncer.go:368`.
   The `hardenshed` doc gains a Charter section.
   The new refusal in `Entry` has no test that reaches it (finding F2).
6. **`BouncerConfig.SeedStencil`/`SeedBrief`, the `seed_stencil` row key and the `defaultSeedStencil` constant.**
   The `NewBouncer` probe and `configRejectUnknown` list are updated.
   The `bouncerEntry` doc is updated.
   `shed-recipe-spec.md` documents only some Bouncer keys individually (`rubric_stencil` and `harden_round` are absent too), so leaving out `seed_stencil` breaks nothing.
   `shedadapters/doc.go` does not name the seed template. Covered.
7. **New stencil `harden-template-seed`.**
   It is added to the `stencils.go` embed and registry, to `roleOpeningStencils["bouncer-seed"]` (so the parent- and edit-directive tests cover it) and to `markers_test`'s marker set.
   The recipe names it, and the rubric's header comment is updated. Covered.
8. **Rubric judge section rewritten: carry the charter forward, never steer outside the operator's focus.**
   Its consumers are the shared `bouncer-template-judge.md`, `RenderJudgePrompt` and the facts writer (`writeRoundFacts` + `Campaign.RoundFacts`).
   None of them was changed to give the judge the charter or the decision record (finding F1).
9. **Rubric review rules gain the Crucible carry-over.**
   Measured against Crucible's README and review-prompt template at `7346d6a8^`, some rules are still missing or contradicted (finding F3).
10. **Recipe review row: target and fasit instructions now point at the focus file and the operator's ask.**
    The burler review prompt reads the focus file through `burler-focus-directive` (`burlerengine/prompt.go:254`). Covered.
11. **`--single-repo` on `lym harden sandbox new`, `SeedLocalOrigin`, `CloneAndWireFromLocalOrigin`, and the `SandboxDeps` doc saying the records bare may be empty.**
    `buildSandboxHub` passes `""` through to `Clone`, which ignores it. Covered.
    `hardencli/doc.go` and the verb's `Long` text are updated, as are `fabricengine/doc.go`, `docs/overview.md` and the integration tests. Covered.
12. **Commit messages, departures and declared PATTERN changes.**
    No `Pattern-Departure`, `Loosened-Check` or `Root-Cause` trailer appears, and none is needed.
    No commit claims an improvement or an agreement.
    The directive declares no PATTERN changes. Nothing to flag.

**Findings**

- **F1 — BLOCKING, design.**
  `contracts/stencils/harden/harden-rubric-review.md`: "The judge" bullets "Carry the charter forward" and "never steers a round outside the operator's focus", and the Scope bullet "no focus file steers a round outside it" (diff 002, lines 27-29 and 129-132).
  The rubric now tells the judge to restate the charter's standing sections and to keep every round inside the decision record's focus.
  The shared judge template it is put into reads only the facts file, the latest review and the previous ledger, "and never the artifacts".
  `RenderJudgePrompt` passes no current-round focus path and no decision-record path.
  The facts file carries neither:
  - `bouncer-template-judge.md`, "Reading order (read all three…)" and "you do not open them";
  - `internal/shedadapters/bouncer.go:2016-2031`, the `judgeValues` map;
  - `internal/hardenshed/campaign_rows.go:168-208`, which writes the models, live, strays, commits and deferred findings only.
  So round 2's focus file cannot carry the charter forward: the layouts, the cost declaration's execution ban and the teardown check are all lost.
  The judge also never sees the operator's focus.
  The decision ("Give the seed and judge calls … their own stencil text or rubric where the shared Bouncer seed stencil cannot carry this") required one of these:
  - a judge stencil or facts section that names round N's focus file and the decision record;
  - or a `RoundFacts` line pointing at both.
- **F2 — BLOCKING, design (`PATTERN-refusal-way-forward`).**
  `internal/loomcli/hardenwiring.go:179`.
  The new refusal "the board holds no entry %q; way forward: name the modules … hardener.packages, then run \"lym loom resume\"" sits in a bound module (`internal/loom*`).
  No test reaches it or asserts its message.
  `git grep "board holds no entry"` finds only production sites, and the reach report lists `hardenCampaign` as reached by no mapped test.
  The pattern requires such a test (`pattern/PATTERN-refusal-way-forward.md`, third bullet).
- **F3 — MEDIUM, design (Stencil wording against the decision "Carry over every rule of the template and README that Hardener's rubric and stencils lack").**
  `contracts/stencils/harden/harden-rubric-review.md`. The rubric still lacks or contradicts these rules:
  - **Merge-bar calibration.** "a timeout under an artificial N-suite CPU peg is not a defect" (README lines 213-219; template lines 854-855). The rubric does not carry it.
  - **The safety pass's "do not invent work".** "'No new defects, ship it' is the expected outcome" (template line 851). The rubric's safety-pass bullet does not carry it.
  - **The pre-count.** Crucible keeps it "into a file the round never sees" and records its blind spots (README lines 359-371). The rubric's "Explain any difference from a count the focus states" shows the count to the round, and it drops the blind-spot rule.
  - **Proof of determinism.** "Prove determinism by running the new test many times in parallel under load" (template line 1002). The rubric does not carry it.
  - **Doc updates.** Update `docs/overview.md` and `PATTERN.md` in the same change when invariants move (template lines 1009-1010). The rubric names only the package doc.
- **F4 — NIT, consistency.**
  `internal/preflightshed/preflight_test.go:172`.
  The fixture error still reads "hardener campaign: the hub's gate.yaml sets no gates.harden-live key", a refusal the diff retires.
  The test passes, but the fixture now cites a message no code produces.

**Tool calls**

1. `grep` of `contracts/specs/shed-recipe-spec.md`, `internal/shedadapters/doc.go` and `internal/shedrecipe/doc.go` for Bouncer key and seed-template mentions: the manifest did not cover the docs that describe the new `seed_stencil` key.
2. `sed`/`grep` of the `internal/shedadapters/doc.go` focus-file section, of stale harden references in package docs and patterns, and of stencil-family tests: I needed to check whether the docs describe the seed template or the hub-writes rule.
3. `grep` for docs naming `carry_over`/`commit_seam`: I needed to find where Bouncer row keys are documented.
4. `grep` of `shed-recipe-spec.md` for its Bouncer key entries: I needed to confirm whether the spec lists every key, which would make `seed_stencil` a forced doc edit.
