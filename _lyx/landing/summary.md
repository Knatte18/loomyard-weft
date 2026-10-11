# Burler: split review and fix rows, per-perch budget and auto-accept

A Burler round was one seat table: the fixer spawned first and waited for the reviewer, so the fixer's orientation was paid for even when the review came back empty.
Both seats also loaded the same rubric, fasit, PATTERN overview, focus file and plan cards.
This change splits the round into two Shed rows, makes the fixer lean, and lets each perch end on its own budget.

## What changes

- **Two rows per perch.**
  Each `BurlerRound` row in `contracts/recipes/loom-recipe.yaml` (Discussion, Plan, Webster) and `contracts/recipes/darn-recipe.yaml` becomes `<Perch>-Burler-Review` then `<Perch>-Burler-Fix`.
  The Bouncer's `on_stuck` names the review row, the review row's names the fix row, and the fix row's names the Bouncer.
  Neither row returns Done, since a Done would end the segment's bounce episode and reset the Bouncer's budget every round.
  Row names are resume identities: a run parked on a retired `*-Burler` row is restarted, not migrated.
- **No fixer on an empty review.**
  With zero findings of any severity (a NIT spawns the fixer), the fix row runs its gates over the unchanged target.
  If they pass, it spawns no session and writes a Go-authored fixer report that parses like any other.
  If they fail, for example after an earlier fix gate was exhausted and left edits in the target, it spawns the fixer on the gate's findings.
- **Lean fixer.**
  Its prompt holds the review, the target paths the findings name, the write surface and the fixer-report rules, and none of the rubric, fasit, PATTERN overview, focus file, prior rounds or plan cards.
  The reviewer-fixer seat channel and the ready marker (`internal/burlermarker`) are removed.
- **The review carries the fix.**
  Every finding gains a `fix` key.
  The review gate refuses a current-round finding with an empty `fix`, a prose finding block with no matching `findings` entry, and any top-level frontmatter key besides `verdict` and `findings`.
  Prior-round hydration keeps the tolerant decode.
- **Per-perch budget and auto-accept.**
  `loom.yaml` gains `discussion_`, `plan_` and `webster_` `max_bounces` (0 takes `review_max_bounces`) and `auto_accept` (false by default).
  A negative key or a resolved budget below 1 is refused naming the key.
  With auto-accept on, a spent budget with no recorded decision is settled by the Bouncer as an accept: the open findings are carried into `## Open risks`, no escalation or parent notice is written, and the run continues.
  Defaults keep today's routing, budgets and escalations.
- `lyx burler run` runs the reviewer, then the fixer only when the review has findings.

## Design decisions

- Each row runs a one-seat `seatengine` table under the Burler adapter in `internal/shedadapters` (`burlerreview.go`, `burlerfix.go`), not `MultiLLMProducer`, because that type maps success to Done and the rows need Stuck hand-offs.
- Go writes an acceptance record when the review passes its gate.
  It holds a digest of the accepted review, and every round archive moves it with the review.
  A review on disk without one is an orphan and is re-reviewed; with one, a restart resumes at the fix row without re-running the reviewer.
- Auto-accept fires only for the budget cause, and only on a verdict the judge wrote in the same Bouncer call.
  A replayed earlier CONTINUE, a CIRCLING verdict below the budget, a recorded escalation, a recorded decision and a failed bounce seam all behave as today.
- A budget of 1 with a fix gate exhausted in round 1 leaves no judged round, so the run halts for a human whatever the auto-accept flag.
- `pattern/PATTERN-review-round.md` is amended: `premise false` now means false against the code the fixer reads, since the fixer holds neither fasit nor rubric.
- Darn gets the split but no auto-accept key; `darn.yaml` keeps its own `review_max_bounces`.
- Docs and specs change with the code: the `doc.go` of `burlerengine`, `burlercli` and `shedadapters`, both recipe headers and the `loom.yaml` template comments.

## To check by hand

- **Vague `fix` text.**
  A review whose `fix` instructions are vague leaves the lean fixer guessing.
  After an auto-accepted single Webster round, only verify checks the result.
- **No PATTERN overview for the fixer.**
  A code fix may depart from a PATTERN rule that no test pins and the reviewer did not name.
- **Fasit-contradicting finding under auto-accept.**
  A fasit-contradicting or rubric-excluded finding the fixer applied in the settling round is committed, and `## Open risks` carry-over does not record it.
- **Recorded-decision and escalation bounds.**
  Check that the auto-accept rules for a recorded decision or escalation in `internal/shedadapters/circling.go` and `bouncer.go` match the intended criteria.
- **Dispute-ground removal.**
  Check that dropping the fasit and rubric as fixer dispute grounds in `pattern/PATTERN-review-round.md` is bounded as the amendment states.
- **Test helpers in the prompt tests.**
  Check which helpers the prompt tests in `internal/burlerengine` share.
- **`validateFix` refusals.**
  The review gate's `fix`, prose-block and frontmatter-key refusals may lack individual tests.
- **`loomcli` test budget.**
  Check the `loomcli` row in `cmd/lyx/testdata/test-budget.yaml`.
- **Not run by hand.**
  No real-engine run of the new rows was made.
  The hub's own `loom.yaml` (`webster_max_bounces: 1`, `webster_auto_accept: true`) is untracked per-worktree config and is set after landing.
