```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the amended board entry's goal points and the record's three added entries.

**Decisions:**
- G1: Hardener writes its own charter.
- G2: Sandbox hubs of either layout.
- G3: No hub gate writes.
- G4: Done when.
- A1 (added): The charter step is Crucible's orchestrator fill.
- A2 (added): The charter is the Harden-Bouncer seed pass, not a new step.
- A3 (added): The operator can name the campaign's focus.

**Per change:**
- **C1. Bouncer `seed_stencil` key and `SeedStencil`/`SeedBrief` config.** Sites: `bouncer.go` probe and render, `entries_bouncer.go` parse, reject list and harden wiring, recipe `seed_stencil`, `stencils.go` registry and `roleOpeningStencils`, `markers_test`, `bouncer_seed_test`, `hardenerrecipe_test`. Traces to A2 and G1.
- **C2. New `harden-template-seed.md`.** Traces to A1 (the fill sections), A2 (the seed is the charter), A3 (body focus binds; with no body, the whole of the modules) and G2 (the `## Layouts` section).
- **C3. `harden-rubric-review.md` rewrite.** Traces to A1 (live-driving rules), A2 (judge re-seed) and A3 (scope binds).
- **C4. `Campaign.SeedBrief`, the `Entry` seam and `TestCampaignSeedBrief`.** Traces to G1 (packages, else title and labels).
- **C5. New target and fasit instructions on the recipe review row.** Forced by C2/C3 (the charter moved from the record to the focus file); also A3.
- **C6. Preflight changes.** Covers: `CheckCampaign` dropping the empty-package and `Set` refusals, `LiveKey.Set` removal, `gateslot.HubSetsGate` and its test removed, `hardenLiveKey` simplified, the gate `template.yaml` comment, `TestCheckCampaign` rows, `TestHardenPreflight`. Traces to G3 and G4 (Preflight half).
- **C7. Live gate with no packages.** No copies, the key's own scope, and green on a pass that started no test binary, plus `TestLiveGate` rows. Forced by C6/G1, since an empty package list is now legal.
- **C8. `lym harden sandbox new --single-repo`.** Covers `hardencli/cli.go`, `fabricengine.SeedLocalOrigin`, `fabriccli.CloneAndWireFromLocalOrigin`, `sandbox.go` doc, `TestSandboxVerbs` and `TestSeedLocalOrigin`. Traces to G2.
- **C9. `seedReadmeBare` refactor of `SeedLocalBares`.** Forced by C8.
- **C10. Docs.** Covers `docs/overview.md`, the `hardenshed`, `hardencli` and `fabricengine` `doc.go` files, the `hardenshed` `template.yaml` packages comment and the recipe header comments. Forced by C1–C8.

**Per decision:**
- G1: C1, C2, C4, C5. The judge's steering half is unbuilt in effect (F1).
- G2: C8, C2.
- G3: C6. The copies-tier half is built neither way the entry offers (F2).
- G4: C6. The live half is admitted unverified in `summary.md`; that is a run outcome, so no finding.
- A1: C2, C3. Built in part (F3).
- A2: C1, C2, C3. The judge half is unbuilt in effect (F1).
- A3: C2, C3. The judge half is unbuilt in effect (F1).
- No unrequested scope found.
- No `Pattern-Departure:` trailers in the range.

**Findings:**

**F1 — BLOCKING, scope.**
- Location: `contracts/stencils/harden/harden-rubric-review.md:122-132`, with `internal/shedadapters/bouncer.go:2021-2033`.
- Summary: A2 and A3 ask the judge's focus file to re-seed from the charter, carry it forward and never steer outside the operator's focus. The diff puts those rules only in the rubric's judge section. The judge's inputs name neither the charter (the round's focus file) nor the decision record, so it cannot restate what it never reads.
- Effect: round 2's focus file can drop "what to read", the layouts, the cost declaration's execution ban and the teardown check, and it cannot see the operator's focus.
- Evidence:
  - `bouncer-template-judge.md` "Reading order": it reads only the facts, the latest review and the previous ledger, "and never the artifacts".
  - `bouncer.go:2021-2033`: `judgeValues` carries no current-round focus path and no decision-record path.
  - `campaign_rows.go:168-208`: `RoundFacts` writes the models, live report, strays, commits and deferred findings, and no charter.
- Context: A2 said to "give the seed and judge calls … their own stencil text or rubric where the shared … stencil cannot carry this". `summary.md` "The judge keeps the shared template" is the writer's claim that it can.

**F2 — MEDIUM, scope.**
- Location: `internal/gateslot/template.yaml:26`, with `internal/hardenshed/campaign.go:63-69`.
- Summary: G3 says the copies run "derives its single tier instead of needing a key of its own set by the hub, or the copies key is folded into `harden-live` with a copies count". The diff does neither: `harden-live-copies` stays a key of its own at the template tier.
- Effect: a hub that overrides `harden-live` to its live tier gets copies that run tier 0. That hub still needs a gate write to make copies amplify the live tests, which is the write G3 removes.
- Evidence:
  - `template.yaml:26`, new comment: "… and harden-live-copies at one tier below tier3".
  - `summary.md` "The copies key is not folded or derived" names this as the writer's own design decision, with no parent-recorded entry. The cited obstacle is real (`gatecli/run.go:412`: `--gate` refuses `--tier`), but it does not rule out deriving the tier at the call site, for example a copies call made under a key that takes `harden-live`'s tier.

**F3 — MEDIUM, scope.**
- Location: `contracts/stencils/harden/harden-rubric-review.md:83-86, 107-111, 122-128`.
- Summary: A1 says "Carry over every rule of the template and README that Hardener's rubric and stencils lack." The rubric and seed still lack these rules:
  1. The merge-bar calibration: a timeout under the N× concurrent stress peg is a diagnostic, not a defect (README:213-219; template:854-855).
  2. Safety pass: "No new defects" is the expected outcome, so do not invent work (template:851). The rubric's safety-pass bullet at :127-128 drops this.
  3. Pre-count: a count lives in a file the round never sees, with its blind spots recorded (README:359-371). Rubric :86 has the round explain the difference from "a count the focus states", which exposes the count to the round and drops the blind spots.
  4. Prove a new live test's determinism by running it many times in parallel under load (template:998-1002). Rubric :110 keeps only the wait-with-deadline rule.
  5. Update `docs/overview.md` and `PATTERN.md` in the same commit when invariants move (template:1009-1011). Rubric :111 names only the package doc.
  6. Re-drive every BLOCKING fix live in its strongest mode, such as `--force` (orchestrator-prompt:680-682).
- Evidence: the cited lines of `git show 7346d6a8^:crucible/README.md`, `review-prompt-template.md` and `orchestrator-prompt.md` against the rubric as the diff leaves it (diff piece 002, lines 1-138).

Tool calls: none. The inherited context held every file this lens needed.
