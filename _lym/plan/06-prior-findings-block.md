# Card 6 — The prior-findings block and the reviewer's rules on it

**Edit:**
- `internal/burlerengine#priorRoundsBlock`
- `internal/burlerengine#TestComposePrompt_PriorRounds`
- `contracts/stencils/burler/burler-step-review.md`
- `contracts/stencils/burler/burler-review-format.md`

**Uses:**
- `plan:internal/burlerengine#RoundContext`
- `plan:internal/burlerengine#PriorFinding`
- `internal/burlerengine#composeReview`
- `internal/burlerengine#newComposableProfile`

**Intent:**
D2's block, rendered by Go into round N's `prior_rounds` value, and the rules the reviewer follows on it, outside any engine block so the Claude and Codex reviewers read the same text.

`priorRoundsBlock` keeps today's text when `p.Round` is nil.
With a context it renders instead:
- a line naming round N-1's review and fixer-report paths and, when `ChangesPath` is set, the changes file;
- one entry per `PriorFinding`, `decision`-class ones included: id, severity, class, location, summary and fix, and under it the fixer's disposition;
  for `disputed` the ground, the rebuttal and every evidence item, for the others the reason, and "no disposition" when the fixer gave none;
- a held finding marked as held by the parent, and a `ruling` disposition beside it labelled as the fixer's report of a ruling that reached it, never the parent's answer;
- once, apart from the findings, every added entry's text from `AddedTexts`, or a line saying none was added;
- whether the round is incremental.

`burler-step-review.md`'s Prior rounds section tells the reviewer, on a block:
- to give every listed finding a `prior` entry with D3's statuses and their rules, withdraw only an accepted dispute and answer a rebuttal it rejects;
- to resolve a held finding only by quoting the title of a listed added entry that settles it, and otherwise mark it open and carry it;
- to carry every open finding as one finding with `carries`, keeping its class, keeping or lowering its severity, and keeping its location path or moving it to a file the changes file lists, naming the moved site in summary and fix when there is no changes file;
- in an incremental round, to raise a new finding only at BLOCKING and only on a changed hunk, and to locate a regression a fix caused in an untouched file at the changed line that causes it, naming the broken site in its summary.
The same section keeps the first-round text and the clean-room text for a round with no block.

`burler-review-format.md` states the `prior` key and its statuses, the `carries` key, the carried finding's class, severity and location-path rules, the two statuses of a held `decision`-class finding, D4's bound for an incremental round, and that a location opens with the file's path; its "only keys" rule names `prior` as the third key.

`TestComposePrompt_PriorRounds` (tier0) gains rows composing the review prompt for each of `claude` and `codex`: a context with `ChangesPath` set renders round N-1's review path, its fixer-report path and the changes file's path, and one with `ChangesPath` empty names no changes file; a context whose round N-1 holds a disputed finding renders its ground, rebuttal and evidence beside it; a fixed-differently finding its reason; a held finding with a `ruling` disposition renders the held mark and the fixer's-report label; two added entries render once each; a nil context renders today's text.
The rows assert Go-filled values, never the stencils' static prose.

**ImpactSummary:** One prompt function and two burler stencils; every review seat that composes the review step reads the new block, and a round with no context renders exactly as before.
