# Twill for Plan-Write at opus[medium] — support log

## Interview

Autonomous run: no operator answered; every pick below is the writer's, recorded in the Question ledger.

### Turn 1 — Board and code read

Read the board entry (bench rounds 1-3, Sonnet and Fable benches, the operator's build decision) and the entries that touch it: `seats-replaces-singlellm`, `plan-multi-llm`, `config-layers-run-settings`, `fan-master-preread`, `plan-review-validate`, `discussion-premise-cites`, `discussion-multi-llm-bench`.
Read Discussion twill's landing (`50cf6a425`): `internal/twilldirective`, `internal/loomengine/twill.go`, `DiscussionSpec`, the ready marker and its removal seam, the Plan-Write registry entry, `PlanSpec`, the plan template, the round-2 bench directive and wrapper, the bench's J9 record, `TestEnforcement_FabricVocabulary` and its predicates, `planglyph`'s check list, `planparser`'s `knownNonCardFiles`, and the burler template's lenses and fans.

### Turn 2 — Design settled

- The shared directive stencil differs from the benched plan text in three host-specific places (spawn point, rejection rule, severity rule), and its reviewed-files marker renders a bullet list inside backticks; D4 adds three host values and fixes the rendering.
- The plan template has no write-scope section, so the Plan host renders only the directive.
- The bench's lens-review file was untracked; the lens tuning the board plans needs it durable, so D7 tracks it in the plan directory.
- The deterministic check reads declared names, the plan's only fixed spellings, against the scan's own predicates moved into a production package.

### Turn 3 — Lens review

All six forks returned a report. Duplicates merged.

- board-fit + pattern-fit, MEDIUM, D1 empty `plan_producer` fails load (`config.go:726-730` rejects `""`): fixed, empty or absent reads as `single`, criterion added.
- premises, MEDIUM, template's "no review logic of your own" contradicts twill: fixed, D3's spawn point overrides it for the procedure; `single` bytes kept.
- premises + scenarios + pattern-fit, MEDIUM/LOW, `lens-review.md` reaches webster's fingerprint and the prior-plan block: fixed, D7 excludes it from both, names the enumerations, states the allowlist bound; criterion added.
- premises + scenarios, LOW/MEDIUM, marker removal could land inside the rotator's early return: fixed, D6 removes it unconditionally before rotation; criteria for no-stale-files and dead-writer relaunch added.
- scenarios, MEDIUM, owner-set carve-out unbounded: fixed, D10 states the bound.
- goal-scope, MEDIUM, other PATTERN scans' boundary unstated: fixed, Scope Out names them with the reason.
- goal-scope, MEDIUM, D9 misses test row and subtest names: fixed by stating the bound in D9 and Scope Out and narrowing the Goal's claim; rows live only in prose, which a name check cannot read without false refusals.
- goal-scope, LOW, rejection rule reach differs from the Goal: fixed, D5 cites only BLOCKING and MEDIUM rejections.
- goal-scope + leanness, LOW, Scope Out contradicts D4's Discussion change: fixed, carve-out stated.
- goal-scope, LOW, may reviewers edit `lens-review.md`: fixed by D7's bound (no prompt points at it) and the fingerprint exclusion.
- premises, LOW, D9 placement ambiguous: fixed, appended by the plan index like `card-fabric-reference`.
- premises + pattern-fit, LOW, `weftname` stripping: fixed, D10 moves `bareVocabularyTokenIn`; D9 states the carve-out does not apply to declared names.
- pattern-fit, LOW, D9 host rule refuses a test-file helper the scan admits: fixed, stated in D9's bound.
- pattern-fit, LOW, marker subpath spelling: fixed, D6 derives it from `planparser`'s segment.
- pattern-fit + leanness, LOW, D4 cites `PATTERN-producer-pointer-rule` wrongly and names a rejected alternative: fixed, rationale is drift.
- pattern-fit, LOW, why not `testkit`: fixed, D10's rationale.
- scenarios, LOW, `Type.Method` and malformed handles: fixed in D9 with criteria.
- scenarios, LOW, fork give-up vs writer time: fixed, Open risk added.
- board-fit, LOW, `plan-multi-llm` unnamed: fixed, merged into the producer-migration risk.
- board-fit, LOW, board Open items lose a home at landing: rejected, the board entry's lifecycle is the parent's call; noted in the Question ledger.
- board-fit, LOW, `test-tmp-cleanup` temp-file scan: rejected, the moved predicates take no temp files and `t.TempDir` is the existing convention.
- leanness, MEDIUM, Scope pointer to the unreadable bench file: fixed, points at D8; Notes line cut.
- leanness, LOW, D9/D11 rationales argue rejected options: fixed.
- leanness, LOW, two Open risks not actionable: fixed, forks-inherit risk cut, seats risk merged.
- leanness, LOW, assumptions restate decisions: fixed, one-line pointers.
- leanness, LOW, unchanged-directives constraint: fixed, cut.

## Rejected alternatives

- **A plan-only directive stencil** copying the bench text: duplicates the procedure Discussion already uses, against `PATTERN-producer-pointer-rule`, and drifts.
- **Strengthening Discussion twill with the cited-rejection rule and severity definitions**: unbenched for Discussion, and `board-fit` and `leanness` findings have no code to cite.
- **Spawn guard as shared stencil text**: changes Discussion's prompt for a problem seen only in Plan; it lives in the Plan host's spawn point instead.
- **Untracked lens-review file under `.lyx`** (the bench's place): lost at pair teardown, so the later lens tuning has no record.
- **Lens review in the plan overview**: the overview is parsed and webster-facing; a review record there mixes concerns.
- **A `pattern-scans` lens**: the operator chose a deterministic check.
- **Duplicating the vocabulary predicates in `planparser`**: two copies of the owner set drift; the scan and the check must agree.
- **Checking every declared name outside the owner set regardless of landing file**: refuses names the scan admits, such as a production identifier under `tools/`.
- **`plan_producer` default `twill` in the embedded template**: changes every hub's Plan-Write without opt-in; Discussion's precedent kept `single`.
- **A configurable fan key (`plan_twill_fan`)**: YAGNI; the fan is redefinable in `burler.yaml` as `twill-discussion` is.
- **Writer at `opus[high]` or `sonnet[medium]`**: the benches found no gain at 12% more cost, and more repeated defects respectively.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): template default for `plan_producer`? `single`; this repository opts in through its tracked `loom.yaml`. Resolved, D1 and D11.
- Q2 (auto-pick): reuse the shared directive or a new stencil? Reuse with three host values. Resolved, D4.
- Q3 (auto-pick): keep the bench's "state the cards" line? Yes, as part of the Plan spawn point. Resolved, D3.
- Q4 (auto-pick): where does the lens review go? Tracked `lens-review.md` in the plan directory. Resolved, D7.
- Q5 (auto-pick): does the Plan host render a write-scope paragraph? No; the template has no write-scope section and step 4 already creates the marker. Resolved, D5.
- Q6 (auto-pick): which names does `fabric-vocabulary-name` read? `Create` handle members and `Rename` `New` handles, scoped as the scan scopes their landing file where the plan text determines it. Resolved, D9.
- Q7 (auto-pick): where do the shared predicates live? A new stdlib-only production package in the vocabulary owner set. Resolved, D10.
- Q8 (auto-pick): does this task fix the reviewed-files rendering in Discussion's prompt? Yes; Plan reuses the line and it is broken today. Resolved, D4.
- Q10 (open, for the parent): the board entry's two Open items (`plan_max_bounces` measurement, the wrong-parent-directory miss) are out of this task's scope and have no other board home once this entry closes.
- Q9 (auto-pick): the round-2 directive lives only in the hub prime's untracked `.scratch`, which the plan writer may not read; D5 and D8 carry its host texts and lens texts verbatim. Resolved.
