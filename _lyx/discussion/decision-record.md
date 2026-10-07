# Decision record — card-batching

## Goal

Cut the tokens Webster's implementer forks spend by running small, related cards in one fork.
A Go batchifier sizes each card from the plan's card fields and the disk, keeps large cards alone, and splits the plan's card sequence into contiguous batches with the lowest estimated total cost.
Several named profiles in `batcher.yaml` select between it and the identity batchifier, which stays the default.
The estimator is calibrated offline against the measured token use of past runs' forks before any profile is switched on.

## Scope

In:

- `tools/tokencount` reports each Webster fork's token use and ties it to the card or cards it ran.
- A calibration report: the estimator run on past plans, against the disk as each run found it, set beside each card's measured fork tokens.
- A card-size estimator and a cost-model batchifier in `internal/batcher`: per-card estimate, read-set overlap, hard limits, an exact dynamic program over contiguous segments.
- Named profiles in `batcher.yaml`, `active:` naming one; the identity batchifier available as a profile.
- Webster records the partition once per run in `state.json` and every verb reads it from there, so a mid-run tree change never regroups cards.
- Webster records, per batch, the name of the profile that formed it and its estimate, so a real run's tokens per batch can be set against its estimate.
- Webster runs batches in the batchifier's output order: `SequenceBatches` stops reordering and becomes an assertion.
- The plan spec's `Batchable?` column retired in favour of the batchifier's rule; module docs and `PATTERN-batcher-registry` updated in the same commit.

Out:

- Member-level grouping from glyphs (waits on `plan-glyph-precision`).
- Moving cards next to each other where dependencies allow (reordering); the batchifier only cuts the given order.
- Parallel execution of batches (`webster-parallel-execution`).
- The plan spec and plan template rewrite that states the card-order rule to the plan writer (`webster-delete-ordering` keeps it).
- Switching the default profile away from identity; the operator switches profiles after calibration.
- Any LLM in the batching decision.

## Decisions

1. **Calibration comes first in the card order.**
   The plan's first cards are, in order:
   the tokencount per-fork report;
   the estimator with its size source, the segment-cost function (Decision 5) and the coefficients type with its parse of a named profile's `weights` from `batcher.yaml`, which rejects a missing or negative coefficient;
   the calibration report, which `Uses` both and is tested against a fixture `batcher.yaml`.
   The cost batchifier, `active:` resolution, the template's profiles with their initial coefficients (Decision 8) and every webster change follow.
   The tuning card (Auto-mode assumptions) comes after both the calibration report and the template's profiles, since it runs the report with the template's coefficients.
   Rationale: the operator's stated first step is to measure before grouping; nothing groups cards before the tuning because the cost profile ships inactive and identity stays the default until the operator switches.

2. **Per-fork token report in `tools/tokencount`.**
   Each Webster sub-agent transcript is one fork; the report gives per fork its run, the card ids it ran, message count, peak context and the existing weight figure.
   The fork is tied to its cards by the card file names in the fork's prompt (Webster renders one `<plan dir>/<NN-slug>.md` pointer per card); a fork whose prompt names no card file is listed as unattributed, never dropped.
   Rationale: the prompt carries the card ids directly, which is exact where commit timestamps are only near.

3. **Calibration report reads past runs read-only from git.**
   For each past run it takes the plan from the run's last `loom: plan artifacts for <slug>` weft commit that precedes the run's first Webster fork, and the file and test sizes from the parent of the first commit the run's first batch made on the task branch, the tree that batch started from.
   Git is read through `internal/gitrepo`'s local-read (go-git) side only, never raw git and never a mutating call.
   It runs the same segment-cost function the live batchifier runs (Decision 5), with a git-tree size source instead of the disk, using the coefficients of a profile in `batcher.yaml` that the caller names (parsed by the code Decision 1 places with the estimator), and writes a per-card table of estimate (the card's one-card batch cost), measured weight and their ratio, plus a summary of the fit per run and overall: the median ratio and the spread of the ratios (75th percentile over 25th).
   A card named by several forks takes their summed weight; a card no fork names is listed as skipped with that reason.
   Rationale: past runs ran the identity batchifier, so a past card's fork cost is a one-card fork cost; one shared estimator keeps calibration and live sizing from drifting apart.
   Bound: read-only; a run whose plan commit or base tree cannot be found is listed as skipped with the reason, never guessed.

4. **The estimator is a pure function of cards, a size source and profile coefficients.**
   `internal/batcher` exposes a size-source interface (line count of a file, existence, the test files of a package directory); the live implementation is rooted at a told absolute worktree root, the calibration one at a git tree.
   Per card the signals are: its targets, whether created, edited, deleted or renamed, and its `Uses` entries, each with the line count of its file where the file exists (a file that does not exist yet weighs nothing); and the test files of the directories its targets sit in.
   Glyphs map to files only through the glyph chokepoint and refs are classified only through `planparser`.
   A card yields two outputs.
   Its estimated message count is its messages per target, per test file and per `Uses` entry, times the matching coefficients.
   Its read set is a map of entry to weight: each target and `Uses` file weighs its line count times the context-per-line coefficient, and each distinct directory of its targets adds a package entry and a test entry, each carrying the package weight directly, so two cards editing the same file share more than two cards sharing only a directory.
   Rationale: a fork spends its tokens reading files and running tests, and these are what the card and the disk say before the run.

5. **Cost model.**
   A fork's cost is the sum over its messages of the context each message reads: the start-up context plus everything its cards have read so far.
   For a segment of cards in order, the fork first spends its per-fork messages, each reading the start-up context, and card k's messages (Decision 4's count) each read the start-up context plus the union of the read sets of cards 1..k, so shared entries count once and a later card pays for an earlier card's accumulated context.
   The per-fork messages are paid once per batch; that is what merging saves, against the accumulated context it costs.
   A segment's cost is that sum over Decision 4's two outputs, so a one-card batch costs its own per-fork messages plus its own messages.
   The segment-cost function over cards, a size source and a profile's coefficients is the one function the live batchifier and the calibration report both call; a card's estimate is the cost of its one-card batch, and a batch's recorded estimate (Decision 10) is its segment cost.
   All coefficients (start-up context, per-fork messages, messages per target, per test file, per `Uses` entry, context per line, package weight) are profile parameters, and each enters the cost once: Decision 4 applies the message, context-per-line and package-weight coefficients, this Decision the start-up context and per-fork messages.
   Cost is a sum of per-message context reads: with the initial coefficients a message reading N context tokens costs N, and the calibration card rescales it to tokencount's weight unit (Auto-mode assumptions).
   `alone_above` and `budget` are in the unit of their profile's coefficients, so the card that rescales the coefficients rescales both thresholds by the same factor.
   Rationale: the operator's context-read model plus one addition, the per-fork messages paid once per batch, which is the only term by which merging saves (Open risks); merging pays only when cards share much and both are small, and the model says when.

6. **Hard limits hold whatever the model says.**
   Batches are contiguous segments of the plan's card order, so order and every forward dependency are kept.
   `alone_above` and `budget` both compare cost under Decision 5, a single card's cost being that of its one-card batch.
   A card whose single-card cost exceeds the profile's `alone_above` threshold is a batch of its own.
   A batch of two or more cards holds at most `max_cards` cards and its estimated cost stays within `budget`; a one-card batch is always feasible, so Decision 7's split always has a result.
   Bound: a card over `budget` on its own runs alone, as under the identity batchifier, so no card costs more than today; an `alone_above` above `budget` is accepted and changes nothing for such a card.
   Rationale: a card that cost a fork hundreds of thousands of tokens must never share one; the cap bounds context growth and failure localisation.

7. **Optimal split by dynamic program.**
   Over the card sequence, the minimum total cost over all contiguous splits that satisfy the hard limits, `O(n · max_cards)` segment evaluations; ties go to more, smaller batches.
   Rationale: exact and testable, where greedy merging is order-sensitive.

8. **Profiles in `batcher.yaml`.**
   `active:` names a profile; `profiles:` maps a name to its `batchifier` kind (`identity` or `cost`) and, for `cost`, its `alone_above`, `budget`, `max_cards` and weights.
   The coefficients type and its parse come with the estimator (Decision 1), and its parse already raises the missing and negative coefficient errors; this Decision adds the registry, `active:` resolution and the other load errors, and its card ships the template's profiles after the calibration report.
   The registry maps batchifier kind to a constructor that takes the profile; `Active` returns a configured `Batcher`.
   An empty `active:` and `active: identity` resolve to the identity batchifier even when no `profiles:` block exists, so every existing `batcher.yaml` keeps working.
   The template ships `identity` and a cautious `cost` profile (`max_cards` 3, a low budget, initial coefficients until the calibration card sets them), with `active` left on identity.
   An `active:` naming no profile, an unknown kind or a missing or invalid parameter is a load error, which the Batchifier row already maps to Stuck with its way forward naming `batcher.yaml`.
   A parameter is invalid when a coefficient is negative, `budget` or `alone_above` is not positive, or `max_cards` is below 2 for a `cost` profile; no value of the thresholds makes Decision 7's split infeasible.
   Bound: the recorded partition (Decision 10) is replaced only by a successful first init or rebaseline.
   A load error at any verb refuses before that verb touches state, naming `batcher.yaml` as the way forward; a rebaseline whose profile does not load refuses and leaves state unchanged.
   A mid-run change of `active:` takes effect only at the next rebaseline's tail re-batch, and every begun batch keeps the profile name and estimate recorded with it.
   Rationale: the operator switches between profiles by one key, and identity stays the default.

9. **The `Batcher` interface takes the size source.**
   `Batch` takes the cards and a size source; the identity batchifier ignores the source.
   Webster's callers build the disk size source from the worktree root they are already told.
   Rationale: keeps the batchifier a pure function, testable on fakes, and keeps path derivation out of `internal/batcher`.

10. **Webster records the partition once per run.**
    At first init `run` computes the batches and records in `state.json` the ordered card ids of every batch, with the name of the profile that formed it and its estimate.
    Every verb that forms batches from the plan today reads the recorded partition and maps its card ids onto the current plan's cards.
    `lyx webster validate` before a run, with no state, computes the partition fresh.
    A state written before the partition field existed recomputes as today, so an in-flight run survives the deploy.
    Bound: such a state always batches with the identity batchifier, whatever profile is active, so the fallback only reproduces what that run started under.
    Rationale: sizes read from a tree the batches are changing would regroup cards mid-run and renumber batches; the existing per-call recomputation is only safe because identity ignores the disk.

11. **Rebaseline keeps begun batches and re-batches the tail.**
    On an accepted rebaseline, every begun batch keeps its recorded card ids, profile name and estimate; the cards after the last begun batch are re-batched by the then-active profile against the current tree, each new batch recording that profile's name and its estimate, and the recorded partition is replaced.
    The existing guard that a begun batch's card set never changes stays as is: every current-plan card up to and including the last card of the last begun batch must be exactly the recorded card ids of the batches up to there, in order, or the rebaseline refuses with that guard's card-set-changed refusal, naming the way forward it already names.
    A plan edited outside a rebaseline is refused by the existing fingerprint guard, so the recorded partition is never re-derived from an unaccepted plan.
    Cards added, removed or reordered after that point are accepted and re-batched, each changed plan file named with `--card NN` as the changed-file guard already requires.
    A state without a recorded partition (Decision 10) keeps the identity batchifier for the tail and records no partition, so such a run stays on identity until it ends.
    Bound: only unbegun cards can regroup, and only at a rebaseline the operator or Master triggers.
    Rationale: a one-card edit could otherwise move a DP cut into a begun batch and force `reset --to start`.

12. **The batchifier's output order is the execution order.**
    Batches run in the order the batchifier returns them, which is plan card order; Webster no longer reorders.
    `SequenceBatches` reduces to an assertion over the derived edges (`deriveEdges`): a batch depending on a later batch, or a cycle, is a refusal naming its way forward (fix the plan's card order, then `lyx webster rebaseline --card NN` or a fresh run); cycle condensation and the informational `Cycle` warning go.
    The assertion runs where a partition is computed (first init, `validate` with no state, a rebaseline's tail re-batch) and on the mapped batches at every verb that maps the recorded partition, with the recorded order as the execution order.
    Bound: it lets nothing new through; Plan-Gate's backward-dependency refusal already holds, so the assertion fires only on a plan that bypassed or predates that gate.
    Rationale: operator rule 2026-10-06 (`webster-delete-ordering`): a valid plan runs in card-number order; a reorder would split the batchifier's contiguous segments from the order its cost model assumed.
    This takes over `webster-delete-ordering`'s "Webster drops its own ordering" item.

13. **The plan spec's `Batchable?` column goes.**
    `contracts/specs/loom-plan-spec.md`'s card-type table loses the column and its composition rule, and points to `internal/batcher`'s package doc for how cards group.
    Rationale: no code enforces it, and it says Create and Edit cards never share a fork, which this task reverses on the operator's decision.

14. **Docs.**
    `internal/batcher/doc.go` describes the estimator, cost model, hard limits, DP and profiles; `internal/websterengine/doc.go` describes the recorded partition, the tail re-batch and that batches run in the batchifier's order; `PATTERN-batcher-registry` says `active:` names a profile and the partition is fixed per run; `tools/tokencount`'s package comment covers the per-fork and calibration reports.
    Rationale: CLAUDE.md's docs-in-the-same-commit rule.

## Constraints

- `PATTERN-batcher-registry`: selection stays through `internal/batcher`'s registry and `active:`; the entry text is updated (Decision 14), not departed from.
- `PATTERN-config-strictness`: `Active` keeps `configengine.LoadOrTemplate`.
- `PATTERN-told-geometry` and `PATTERN-cwd-resolution`: `internal/batcher` derives no path and never imports `lyxcwd`; the size source is rooted at a told absolute path.
- `PATTERN-ref-shape-registry`, `PATTERN-glyph-conversion-chokepoint`, `PATTERN-sole-parsers`: card refs are classified by `planparser`, glyphs mapped to files by `glyph.Parse`/`UnitPath`, plans parsed only by `planparser`, in the batcher and the calibration tool alike.
- `PATTERN-gitrepo-client-boundary`, `PATTERN-fabric-git`: the calibration reads git history through go-git local reads only.
- `PATTERN-refusal-way-forward`: any new webster refusal (for example a recorded partition naming a card the plan no longer has, or a rebaseline whose profile does not load) names its way forward, lands its spec row and is reached by a test.
- `PATTERN-no-denied-recovery`: no way forward names a denied command.
- `PATTERN-test-speed`, `PATTERN-test-economy`, `PATTERN-test-isolation`: batcher tests are untagged and pure; git-reading calibration tests use `gitkit.HermeticGitEnv()`.
- `tools/` follows PATTERN and the enforcement scans like every other package.
- Batches stay contiguous in card order, and the batchifier's output order is authoritative; no Webster code relies on `SequenceBatches` to order batches.

## Auto-mode assumptions

- The default profile stays identity; the cautious cost profile ships but is not active.
- The template ships initial cost-profile coefficients and thresholds in context tokens; a late card sets them from the calibration report's per-card table by one rule: a single scale factor, the median ratio of measured weight to estimate, multiplies the start-up context, context-per-line and package-weight coefficients, the only coefficients that carry a context size, so every estimated cost scales by exactly that factor; the card multiplies `alone_above` and `budget` by the same factor and commits all the numbers in the template.
  The per-fork messages stay at their initial value, since past forks are one-card forks and cannot identify them.
  The report runs once on the operator's transcripts; if it cannot be produced or the fit is poor, the card commits nothing and the initial values stay.
  A fit is poor when the spread of the per-card ratios (75th percentile over 25th) exceeds 3; the Plan producer may tune that number.
- The per-fork report ties forks to cards through the card file names in the fork prompt, not commit times.
- The calibration measure is tokencount's existing weight figure.
- Profile parameter names are `batchifier`, `alone_above`, `budget`, `max_cards` and a `weights` map; the Plan producer may rename them for consistency with other config.
- `batcher.yaml` stays a per-worktree config, not hub-wide.

## Open risks

- The estimator may fit past one-card forks poorly; a weak fit means the cost profile's merges are only as good as its weights, so the operator switches it on cautiously.
- The per-fork messages, which are the whole saving of merging, cannot be fitted from past one-card forks; the saving is only as good as their initial value until real runs under a cost profile are measured.
- A merged batch's cost was never observed offline, so the saving is only measured on real runs after a profile switch.
- A failure in a multi-card batch localises less precisely; `recover-batch` restarts the whole batch.
- Drift is checked per batch, not per card; `max_cards` bounds how many cards run between checks.
- Past runs' plan and base-tree lookup in git may miss runs whose weft branch was torn down without an archive tag.

## Acceptance criteria

- `tools/tokencount` lists each Webster fork with its run, its card ids, message count, peak context and weight; a fork with no card pointer is listed as unattributed.
- The calibration report runs on the hub's past runs and writes, per card, the estimate, the measured weight and their ratio, with a per-run and overall fit summary (median ratio and spread); runs it cannot reconstruct are listed as skipped with the reason, a card no fork names is skipped, and a card named by several forks takes their summed weight.
- The live batchifier and the calibration report call the same segment-cost function.
- Batcher unit tests pin on every input: card order kept and batches contiguous; a card above `alone_above` alone; no batch over `max_cards` or `budget`; the DP result is the minimum-cost split (checked against brute force on small inputs); identity yields one card per batch.
- A card whose one-card cost is over `budget`, whether or not it exceeds `alone_above`, forms a one-card batch and the split still has a result.
- Two small cards editing the same file group under a cost profile; two small cards in unrelated packages do not.
- A segment's cost counts the per-fork messages once, so two cards with identical read sets cost less merged than apart, and a card whose read set adds much to an earlier card's context costs more merged than apart.
- `batcher.yaml`: an empty or `identity` `active:` with no `profiles:` block resolves identity; an `active:` naming a configured cost profile resolves it; an unknown profile, unknown kind or invalid parameter (a negative coefficient, a non-positive threshold, `max_cards` below 2) fails to load, and the Batchifier row reports Stuck naming `batcher.yaml`.
- A run under a cost profile records its partition, with each batch's profile name and estimate, at first init; changing a file between batches leaves later verbs on the recorded partition.
- A state without a recorded partition still runs, recomputing batches as today with the identity batchifier even when a cost profile is active.
- A batcher.yaml that fails to load mid-run refuses the verb before it touches state; a rebaseline whose profile does not load leaves the recorded partition and state unchanged.
- Webster runs batches in the batchifier's output order; a plan with a backward dependency or cycle between batches is refused with a way forward, never reordered.
- An accepted rebaseline keeps every begun batch's card ids, profile name and estimate and re-batches only the unbegun tail; a plan that gained, lost or reordered a card at or before the last begun batch's last card is refused; on a state without a recorded partition the tail stays on the identity batchifier and no partition is recorded.
- The plan spec no longer carries the `Batchable?` column; module docs and `PATTERN-batcher-registry` describe profiles and the recorded partition.
- `go test ./...` passes.

## Notes for the plan writer

- Webster already handles multi-card batches: `contracts/stencils/webster/webster-body-implementer.md` asks for build, test, verify and commit per card in declared order.
- Batches are recomputed today in `internal/webstercli`'s verbs and in `websterengine.Run`; `Rebaseline` compares begun batches against the recomputed ones.
- tokencount already groups sub-agent transcripts under `<session>/subagents/` as `<role>+sub`.
- Past plans: the hub's weft repository holds `loom: plan artifacts for <slug>` commits, several per slug; archived weft branches are reachable through `archive/<slug>/<tip>` tags.
