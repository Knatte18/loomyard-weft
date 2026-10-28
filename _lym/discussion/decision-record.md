# Decision record — review-setup-tuning

## Goal

Build the lens and code-review recommendations of `docs/research/review-setup-audit-2026-10-28/report.md` that the operator accepted on 2026-10-28:
one board-and-scope lens in place of `goal-scope` and `board-fit`, a repository-invariant lens in place of `way-forward` in `review-plan`, `code-pattern-fit` folded into `code-affected-set`, a narrower `code-tests`, and a solo code review in the `light` and `docs` profiles, and in `speed`'s loom block.

## Scope

In:

- The lens prose, fans and header in `internal/lenses/template.yaml`, `TestConfigTemplate`'s procedure-lens exemption, and the `review-code` fan of the `csharp` language layer in `internal/lenses/sets/csharp.yaml`.
- The `light` and `docs` profiles and `speed`'s loom block in `internal/runprofile/template.yaml`.
- The config category of loom's `code_reviewer_fan` key, which the profile change needs.
- The docs D7 names.

Out:

- Delta review of later rounds, and round counts per profile (the entry's out-of-scope list).
- New checks in `validate-discussion` and `validate-plan`, which belong to `validator-checks`.
- The `twill-plan` fan, which keeps its five lenses, `way-forward` among them.
- The fixer fans, `code_fixer_fan` included: the entry names the reviewer only.
- The hub's live `profiles.yaml` and `lenses.yaml` on Bolt, which no landing writes (see Open risks).

## Decisions

### D1: one `board-scope` lens replaces `goal-scope` and `board-fit`

Add a procedure lens `board-scope` to the template and delete `goal-scope` and `board-fit`.
`board-scope` takes the place of the pair in `twill-discussion`, giving `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `board-scope`, `leanness`, and replaces `goal-scope` in the dormant `discussion` fan.
`review-discussion` drops the pair and gets no replacement, giving `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`.
Its prose:

- Reads `lym board list`, `lym board get` for this task and `lym loom peers`, then up to three reads, nearest first, of a `Decision for <slug>:` note for this task, a touching entry, a close peer's record or a parent commit `lym loom peers` lists; at most six tool calls.
- Holds a `Decision for <slug>:` note and the record's `Added after Discussion` entries as parent decisions that win over the entry body.
- Reports only a contradiction with the board: a goal point of the entry the record leaves out or defers, work the entry rules out, work another entry owns, and a design conflict with another entry, run or landed parent commit.
- Reports doubt about what the task is for, an entry plainly outdated by landed work, and a conflict the writer cannot settle as a finding of class `decision`, phrased as a question for the run's parent; everything else is an ordinary finding.
- Excludes text-only overlap, shared bookkeeping lines and any question a decision note or `Added after Discussion` entry settles.
- Ends in its list: each goal point met or not, each entry, peer and commit checked with conflict or none, and each decision note read or unreachable.
- Stays a procedure lens though it sits in no `review-*` or `fix-*` fan: the template header's procedure-lens sentence and `TestConfigTemplate`'s emphasis-line exemption widen to name `board-scope`, since an emphasis line inviting anything else would undo its report-only contradictions rule.

Rationale: the audit (§2, twill-discussion) measured the pair as the writer fan's noisy lenses with no BLOCKING item; one lens that reports contradictions only keeps their gating finds at one fork's cost.
The audit (§2, review-discussion) recommends the reviewer fan keep its four lenses, which deleting the pair gives.

### D2: `repo-invariants` replaces `way-forward` in `review-plan`

Add a procedure lens `repo-invariants`; `review-plan` becomes `affected-set`, `guard-reach`, `repo-invariants`.
`way-forward` stays defined and stays in `twill-plan`.
Its prose gains the emphasis line after its list, as `coverage`, `row-validity` and `forced-edits` end, since it leaves the only procedure fan that exempted it.
`repo-invariants`' prose owns the repository invariants a card breaks without touching the line that breaks:

- Test seams: a field a card adds to a shared struct that a reflective seam test must cover, and each new test file's one tier tag.
- Doc text a card falsifies: the package `doc.go`, `PATTERN.md` entries and `pattern/` background files that describe behaviour the card's Intent changes, with no card listing them.
- Card order: a card that cannot pass its own gate from the state the cards before it leave.
- A broken invariant no card lists is a finding; the list is per card, each invariant checked, held or broken.

Rationale: the audit (§2, review-plan) found `way-forward` the weakest plan lens, repeating a writer lens the writer already ran;
these invariants are what Plan rounds 2 and 3 found late in text no fixer touched (§4).
The test that every reviewer lens is in its writer fan gains `repo-invariants` as its one reviewer-only lens, since the entry asks for a lens the writer fan lacks.

### D3: `code-pattern-fit` folds into `code-affected-set`

Delete `code-pattern-fit`; `review-code` becomes `code-correctness`, `code-affected-set`, `code-tests`, and the `csharp` layer's `review-code` becomes those three plus `numerics`.
`code-affected-set`'s set gains the repository's rule documents that describe a changed behaviour: the `PATTERN.md` entries and `pattern/` background files where the repository has a `PATTERN.md`, otherwise the rules its `CLAUDE.md` files state.
It applies, where the round's rubric carries them, "Stencil wording", "The repository's own rules", "Declared PATTERN changes landed" and "Commit messages and departures".
The `code-pattern-fit` prose also named "A broken invariant" and "A pattern the parent changed", which no rubric carries; those names are dropped.
Rationale: the audit (§2, review-code) found most gating `code-pattern-fit` findings to be stale prose a change left behind, an affected-set question; the rubric bullets it applied move with it.

### D4: `code-tests` owns missing coverage of changed behaviour only

`code-tests`' prose becomes: for each behaviour the diff adds or changes, find the test row that would fail if it were broken; a behaviour with no such row is a finding, a stencil's static prose exempt; the list is per behaviour, the test that pins it, or none.
The lens no longer opens the existing tests of each new row and no longer applies the rubric's "Test economy".
The code-review rubric is unchanged, so the review lead still judges test economy.
Rationale: the audit (§2, review-code) found `code-tests` the noisiest code lens, its noise test style and redundancy.

### D5: `code_reviewer_fan` opens to the local layer

Reclassify loom's `code_reviewer_fan` from `CategoryGuard` to `CategoryNone`, so a profile block, a task's `run_settings` and a pair's local layer can set it.
The other five reviewer and fixer fan keys stay guards.
The retired darn keys `darn_reviewer_fan` and `review_fan`, which `configreg`'s key move carries to `loom.code_reviewer_fan`, are admitted with it, and so are loom's retired `webster_reviewer_fan` and `webster_fan`, which loom's `MigrateConfig` renames onto `code_reviewer_fan` before the check in `PrepareRunSettings` and in a reconcile of the local file; a hand-edited local file holding one is refused at load as a key outside the loom template, since neither is declared one of its retired keys, until a reconcile migrates it; the same bound holds for every one of these routes.
Bound:

- A value can name only a fan the guarded `lenses.yaml` defines, or `""` for a solo round; the write is not checked against `lenses.yaml`, and a wrong name fails the pair's loom config load through `ValidateFanKeys`, whose refusal names the key and the fix.
- The perch's round count, its rubric and the reviewer's read of the whole diff are unchanged.
- An agent can thin its own run's code review to solo, or point it at any other fan `lenses.yaml` defines, through its pair's local layer; either way the round keeps the reviewer's whole-diff read under the code-review rubric, which is the floor; it can already turn the perch off from the same file with `code_max_bounces: 0`, which stays the stronger switch.
  A task's `run_settings` reach a pair only at Seed-Child, so a `run_settings` write during a run changes no running pair.

Rationale: Seed-Child writes a profile block into the pair's local layer, and configreg checks a profile block as it checks `run_settings`, through `checkLocalDocument`'s `configengine.ValidateLocal`, which refuses a guard key;
without the reclassification no profile can make the code review solo.

### D6: `light`, `docs` and `speed`'s loom block run a solo code review

In `internal/runprofile/template.yaml`, the `light` profile's `loom`, `darn` and `pawl` blocks, the `speed` profile's `loom` block and the `docs` profile's `darn` block each set `loom.code_reviewer_fan: ""`.
`speed`'s `darn` and `pawl` blocks set `code_max_bounces: 0`, which runs no code round, so they set no fan; `solid` and `wide` set no fan either, so they take the hub's `code_reviewer_fan`, which is the template's `["review-code", ""]` where no layer sets the key.
A run already seeded keeps the block it was seeded with.
Rationale: the entry's profile line; the audit (§1) found the code fan's first rounds mostly empty of gating findings.

### D7: docs land with the change

- `internal/runprofile/doc.go` states that `light`, `docs` and `speed`'s loom block run the code review solo.
- Every doc and comment that calls all reviewer fan keys closed to the local layer names `code_reviewer_fan` as open: `docs/overview.md`'s loom config line and loom's `config.go` comments.
- `docs/dotnet-hub.md`'s `review-code` example drops `code-pattern-fit`.
- `pattern/PATTERN-config-layers.md`'s Guard bullet names `code_reviewer_fan` as the one reviewer fan key open to the local layer, with D5's bound; the `PATTERN.md` overview line is unchanged.

Rationale: CLAUDE.md's docs-in-the-same-commit rule.

## Constraints

- `PATTERN-config-layers`: D5 departs from the background's Guard text, "the prose or lens set a reviewer is given", for one key; the background changes in the same commit (D7), and the classification test keeps every template key classified.
- D6's value survives reconcile: `configsync` reconciles a pair's local file with no strip, so its former-template-value strip, which lists `code_reviewer_fan: ""`, never removes it there.
- `PATTERN-sparse-hub-base` and the lenses reconcile: a hub's own `lenses.yaml` keeps only tuned entries, so template edits to the changed lenses and fans reach every hub that did not tune them.
- `profiles.yaml` is seed-only: the template edit reaches only a hub seeded after the landing.
- The lens prose follows the template's rules: procedure lenses, those of the `review-*` and `fix-*` fans and `board-scope`, end in their own list, every other lens ends with the emphasis line, and no lens tells a fork to ignore anything.
- Every lens and fan this task changes lives in `internal/lenses/template.yaml` and `sets/csharp.yaml`; no file under `contracts/stencils` names a changed lens.
- `validator-checks` (running) adds machine checks for seam tests, tier tags and `doc.go` lines; `repo-invariants` names no check of its, so the lens stays right whether or not those checks land first.

## Auto-mode assumptions

- D1: the merged lens also replaces `goal-scope` in the dormant `discussion` fan, which the entry does not name.
- D1: `review-discussion` drops `goal-scope` and `board-fit` with no replacement, which the entry does not name.
- D3: the deleted lenses go outright, with no alias or retired-name handling.
- D6: `light`'s loom block runs the code review solo too, beside its darn and pawl blocks.
- D6: `speed`'s loom block runs the code review solo, which the entry does not name, so the lightest profile never fans a code review that `light` runs solo.

## Open risks

- The hub's live `_board/_lym/config/profiles.yaml` is seed-only and operator-owned, so D6 does not reach it: the operator re-sets `light`, `docs` and `speed`'s loom block with `lym config profiles --set` after the landing deploys.
  On this hub the live `docs` block sets `code_max_bounces: 0`, so its code review is off and the fan setting is moot until that changes; the live file also has no `pawl` blocks.
- A hub whose `lenses.yaml` defines a fan naming `goal-scope`, `board-fit` or `code-pattern-fit` fails once the deploy lands: a fan one of the six reviewer or fixer fan keys names fails every loom config load, since `LoadConfig` resolves those keys through `ValidateFanKeys`, and every loom run on that hub stops at its first step; a fan named only by `discussion_lenses` or `plan_lenses` fails later, when the Discussion-Write or Plan-Write lead resolves it.
  `docs/dotnet-hub.md` shows the OSL-1046 Models hub with such a `review-code` fan; its operator drops the name from the fan, with `lym config lenses --set` or the editor, before the deploy.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - d2-way-forward-loses-emphasis-exemption (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: `twill-discussion` resolves to `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `board-scope`, `leanness`; `review-discussion` to `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`; `discussion` names `board-scope` in place of `goal-scope`; the template defines neither `goal-scope` nor `board-fit`.
- D1: `board-scope`'s prose caps its reads at six tool calls, reports only contradictions with the board, phrases a parent question as class `decision` and ends in its list; `TestConfigTemplate` exempts `board-scope` from the emphasis line beside the `review-*` and `fix-*` lenses, and still fails any other lens of a non-procedure fan, `leanness` among them, that does not end with the line.
- D2: `review-plan` resolves to `affected-set`, `guard-reach`, `repo-invariants`; `twill-plan` is unchanged and `way-forward` stays defined and its prose ends with the emphasis line.
- D2: the writer-covers-reviewer check passes with `repo-invariants` as the one reviewer-only lens, and still fails for any other reviewer lens missing from its writer fan.
- D3: `review-code` resolves to `code-correctness`, `code-affected-set`, `code-tests`; under `lens_set: csharp` to those plus `numerics`; the template defines no `code-pattern-fit`.
- D3: `code-affected-set`'s prose names the repository's rule documents and applies "Declared PATTERN changes landed" and "Commit messages and departures".
- D4: `code-tests`' prose names no "Test economy" and keeps its per-behaviour list.
- D5: a local layer or a task's `run_settings` holding `code_reviewer_fan`, or a retired darn key or loom's retired `webster_reviewer_fan` or `webster_fan` moved onto it, is admitted; one holding any of the other five fan keys is still refused.
- D6: the template's `light` `loom`, `darn` and `pawl` blocks, `speed` `loom` block and `docs` `darn` block carry `code_reviewer_fan: ""`, every block passes configreg's check, and `speed`'s `darn` and `pawl` blocks, `solid` and `wide` carry no fan key.
- D6: a pair seeded from a `light` profile loads `code_reviewer_fan` as one empty entry, and its Code-Review round runs as a solo round, not a fan.
- D7: the named docs and comments state the new behaviour.
