## Interview

Autonomous run: no operator interview.
Questions only the parent could settle went to the orch through creel.

### Turn 1 — orch, unprompted (creel #156)

The orch relayed two operator decisions: the arbiter, tackler and driver-kind keys leave `loom.yaml` and `darn.yaml` for `shed.yaml` alone, a recipe's own values going in its profile block; and `lym loom` and `lym shed` both stay, `lym shed` serving every recipe.
These settle the entry's first and last Open items and became D13 and D14.

### Turn 2 — question to the orch (creel #158), answered (creel #159)

Asked: batten's Run-Shed starts pawl with `lym loom start` and the orch drives beams through `lym loom` verbs, while the entry restricts `lym loom` to loom recipes and leaves pawl alone.
Recommended: pawl keeps its `lym loom` verbs as one named exception until `pawl-loom-recipe` removes it.
Alternative: refuse pawl now and give `lym shed` a start and resume, which touches pawl.
The operator chose the recommendation; it became D2.

### Lens review

Six forks ran, and every one returned a report.

- interactions 1 / existing-paths 1 (BLOCKING), `decision add` fails on a darn record with no support log: fixed, D4 now writes a support log too, so the append path validates the pair unchanged.
- walkthrough 1 / existing-paths 2 (BLOCKING), removing the board-entry snapshot breaks pawl's perch: fixed, D7 keeps the snapshot writer for pawl and stops only darn writing it.
- walkthrough 2 / interactions 2, a no-parent writer stop no longer halts: fixed, D6 halts on a Darn-Write handoff with no commit past the fork.
- interactions 3 / bound-breaker 1, in-flight darn runs with no record: fixed, D4 halts them at Darn-Write naming `goto --to Record-Write`, and lists them under Open risks.
- interactions 4, pawl's verify source and a missing `verify_source`: fixed, D1 requires the key on a loom recipe and D8 keeps pawl's source under D2.
- interactions 5, no criterion for the writer applying PATTERN bullets: fixed in D3's criterion.
- walkthrough 3 / existing-paths 3, darn's Drift rows read the board entry: fixed, D4 switches them to `subject: decision-record`.
- walkthrough 4, `decision add` before Record-Write: fixed, D4 names the existing missing-record refusal.
- existing-paths 4 / bound-breaker 4, removed keys in profiles, run settings and the per-recipe override: fixed, D13 refuses them there with a way forward and removes the override read.
- existing-paths 5, the `plan_max_bounces` fallback crossing modules: fixed, D12 states it.
- existing-paths 6, setting references beyond `entry_gate`: fixed, D12 covers every `module.key` reference.
- existing-paths 7, a second Drift-Verify on reroute: fixed, D9 states it is valid, citing `internal/driftengine/doc.go`.
- bound-breaker 2, a run inside the perch skips Drift-Verify: fixed, D9 states it and names the `goto`, and Open risks lists it.
- bound-breaker 3, the `beam` refusal's wording: fixed, D11 names both refusals.
- bound-breaker 5, peers on another machine have no seed: fixed, D3 falls back to the entry's `recipe` field.
- bound-breaker 6, in-flight darn runs reviewed against a writer-written record: fixed, D7 states it.
- bound-breaker 7, the `CheckSeed` claim was not checked: fixed, D10 cites its doc and names what goes.
- board-scope 1, whether `KeyMoves` rewrites `profiles.yaml`: fixed, D12 says profiles are translated on read and never rewritten.
- board-scope 2, `batten.yaml`: rejected, already under Open risks.
- leanness 3, a settled seed note listed as a risk: fixed, moved to Constraints.
- leanness 5, the plan-writer note contradicting D2: fixed, note dropped.
- leanness 6, the Scope In list restating decisions: fixed, it now points at D1 to D14.
- leanness 1, 2, 4 and 7: rejected; the section is required, or the line is a short rationale or pointer the reader uses.

## Rejected alternatives

- Contract check in `internal/loomrecipe` rather than Shed's loader: the entry says Shed checks the recipe when it loads it, and a check outside the loader would miss a recipe built another way.
- Contract expressed as roles each registry entry declares: more machinery than three engine sets beside the registry need.
- Recipe-name lists for verb applicability (D3): a future loom recipe with a plan would need code changes; row presence needs none.
- Mechanical record written at seed time or inside Preflight: a seed carries no board body, and Preflight is shared with pawl; a row of its own is resumable and named in the run's history.
- A record with no support log and a validator that skips the support-log check for a mechanical record: `decision add`'s append validates the record pair through `internal/discussionparser`, so it would fail on every darn run (lens review); D4 writes a support log instead.
- Darn-Write writing the mechanical record itself when it is missing, for in-flight runs: two writers of one file; a halt naming `goto --to Record-Write` uses the existing verb.
- Ignoring a retired arbiter or driver-kind key in a profile: a run pinned to the llm driver would silently run go.
- Removing the board-entry snapshot writer outright: pawl's perch reads it until `pawl-loom-recipe`.
- Outdated-entry check as its own row or in the mechanical row: judging it needs an agent reading code and board, which only the writer already is.
- `batcher.yaml` load in the Webster row: it would fail only after Discussion and Plan spent their tokens.
- Moving the arbiter, tackler and driver-kind keys into `shed.yaml` with `KeyMoves`: a loom-only override would silently apply to every recipe.
- No `KeyMoves` for the `plan_*` split: a hub that tuned a plan key would lose it on reconcile.
- Verify source chosen by deriving from the rows (a recipe with Plan-Write uses the plan): implicit, and a darn-like recipe with a plan row would be misread.

## Review rounds

_No rounds yet._

## Question ledger

- Where `arbiter*`, `tackler*` and `driver_kind` live — resolved by parent decision (creel #156): `shed.yaml` alone (D13).
- How `lym loom` relates to `lym shed` — resolved by parent decision (creel #156): both stay (D14).
- Whether pawl keeps the `lym loom` verbs until `pawl-loom-recipe` — resolved by parent decision (creel #159): yes, one named exception (D2).
- Whether `validate-discussion` accepts a record whose sections bar `## Goal` are empty — resolved by research: `internal/discussionparser` has no empty-section finding, so it does (D5).
- Where the outdated-entry check lives — auto-pick: darn's writer, answering through the parent and `decision add` (D6).
- Which suspected verb gaps are real — resolved by research: plan verbs and `peers` are real, `--amends-plan` already refuses on darn, `--pattern-*` is meaningful on darn (D3).
- Name and place of the mechanical record row — auto-pick: `Record-Write`/`RecordWrite`, after Preflight (D4).
- History check on `tapestry` records — auto-pick: yes, one validator for every loom record (D5).
- How the verify source is chosen — auto-pick: a recipe-file `verify_source` key (D8).
- Where the `batcher.yaml` load goes — auto-pick: Preflight, for a recipe with an Implementer row (D10).
- Which shared keys stay in `loom.yaml` — auto-pick: the review, judge, seed, driver-model and step-idle keys every loom run reads (D12).
- How a tuned `plan_*` key reaches `tapestry.yaml` — resolved by research: `KeyMoves` in `internal/configreg/keymove.go` is the declared route (D12).
- Whether `batten.yaml`'s overrides go too — auto-pick: no, the parent named `loom.yaml` and `darn.yaml`; raised under Open risks (D13).
- Retire or move the removed arbiter keys — auto-pick: retire, refused in profiles and run settings (D13).
- How a writer stop halts once the record is mechanical — auto-pick: a handoff with no commit halts (D6).
- What in-flight darn runs with no record do — auto-pick: halt naming `goto --to Record-Write` (D4).
- Whether Tapestry-Preflight's seed check is redundant — resolved by research: `CheckSeed`'s doc says Shed's step-1 read pre-empts four of its verdicts (D10).
- Whether Drift-Verify tolerates a second pass — resolved by research: it compares the HEAD it meets and is Done on a pass (D9).
