# Per-role model keys, CONVERGED guard, judge and seed usage

Each Bouncer row's judge and seed pass, and PR-Rework, now take a `loom.yaml` model key of their own.
Go also stops trusting the judge alone on convergence, and judge and seed passes are recorded in the per-round usage files.
With every new key empty or absent, every role runs on the model it runs on today.

## Model keys

- `loom.yaml` gains `discussion_judge`, `plan_judge` and `webster_judge`.
  Empty takes the run-wide `judge`; a set value replaces it for that segment's Bouncer row only.
- `loom.yaml` gains a run-wide `seed` and per-segment `discussion_seed`, `plan_seed` and `webster_seed`.
  A segment's seed resolves to its per-segment key, else `seed`, else that segment's resolved judge model.
- `loom.yaml` gains `rework`; empty takes `plan`.
  PR-Rework keeps `plan_timeout_min` as its timeout.
- Every key is parsed at config load and resolved at run start through the model registry, so a bad alias fails before round 1 with an error naming the key.
  The resolved settings reach each Bouncer row through `Env.RowBouncerModels`, keyed by segment; a row whose segment has none is a wiring error at recipe build.
- The Bouncer recipe entry no longer accepts `model`, `effort` or `version`; a recipe naming one is rejected as an unknown key.
  No embedded recipe set them.
  `Env.JudgeModel`, `JudgeEffort` and `JudgeVersion` are gone, and `docs/research/haiku-judge.md` is updated to match.
- `RowBouncerModels` is listed as a legal nil seam in `envkit`, since batten wires no Bouncer rows.

Out of scope, per the board entry: Webster's implementer and verify-fix forks, the Bouncer escalation fork and the fan's lens forks (a fork runs on its parent's model), `landing.yaml` `describe`, `darn.yaml` `writer`, and any Haiku move for the judge or seed.

## Convergence guard

In the Bouncer's harvest of a judge pass:

- A CONVERGED verdict is refused when the round's ledger has an entry that is `open` and `BLOCKING`, or when the round's review report carries a BLOCKING finding or does not parse through the facts file's review parser.
- Any verdict of a round 2 or later judge is refused when its ledger lacks a key from round N-1's ledger.
  The same check runs when Go settles a CONTINUE or CIRCLING verdict already on disk at Call entry with no live judge.
  An unreadable previous ledger skips the check with a logged warning.
- A CONVERGED verdict on disk at Call entry with no live judge is never settled: Go clears the generation and re-seeds.
- A refusal takes the retired-verdict route: Go archives the round's verdict, ledger and focus files, and returns a degraded Stuck whose Reason names the cause and ends in `way forward: the round is re-judged`.
  Each refusal spends one bounce unit, so repeated refusals reach `review_max_bounces` and block the run with Shed's budget-exhausted halt.
- `contracts/specs/refusal-spec.md` has the shed-section row for this refusal.

## Usage records

`round-<N>-usage.yaml` gains optional `seed` and `judge` lists, each entry in the existing per-half shape.
A judge pass appends to round N's `judge`; a seed pass appends to round 1's `seed`.
Each writer merges only its own part: the Burler round writer keeps `seed` and `judge`, and the Bouncer keeps the rest.
A malformed existing record is archived and replaced by one holding only the writer's part, with a logged warning.
A failed usage write only logs.
The lens-fact and lens-exclusion readers treat a record with no review half as missing.

## Design decisions to know

- A refusal never escalates to the parent.
  On a round whose budget is already reached, the single refusal blocks the run with the budget-exhausted halt and no re-judge, where a valid CONTINUE would escalate.
  The operator recovers through the block's `lyx shed goto` way forward.
- The review-side check reads the latest round's review report only, not the fixer report.
- The key check compares keys only, not status, class or severity.
- A pending escalation accept recorded by the parent reaches Done without the guard, open BLOCKING findings included.
- The merge-on-write usage record relies on one segment's Bouncer and Burler rows never running concurrently.

## To check by hand

- `replay-guard-exemption-keys-on-escalation-only` (design, MEDIUM, `internal/shedadapters/bouncer.go`): the replay-path guard exemption keys on a recorded escalation alone.
  Check that no settle path other than a parent-recorded escalation accept skips the guard.
- Run a real loom with `plan_judge` and `webster_seed` set to confirm the spawned sessions' models; the tests pin resolution with fakes only.
- Docs touched alongside the code: `internal/loomengine/template.yaml` key comments, the roles section of `contracts/specs/llm-model-spec.md`, the `internal/shedadapters` package doc and the loom recipe's Bouncer-row comments.
