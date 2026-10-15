# Card 6 — Prune the loom, alignment and darn prompt tests

**Edit:**
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestReworkTable`
- `internal/loomengine#TestParentReviewDeliveryPrompt_OneLineNamingReviewerSlugBrief`
- `internal/loomengine#TestParentReviewBrief_NamesPathsAndSubmitLines`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/alignment#TestPrompt`
- `internal/darnengine#TestDarnSpec`

**Uses:**
- `internal/loomengine#planTwillSeverityRule`

**Intent:**
Prune the loom producers' prompt assertions by D1, keeping Go-filled values (paths, slugs, the twill host values `internal/loomengine/twill.go` fills), the anchors of Go-chosen fragments, the heading-order checks of Go-filled markers, the leftover-marker checks and the `unboundedAskOperator` scan:

- `TestPlanTable`, twill subtest: drop `subagent_type: "fork"`, "your context holds the decision record", "against the code and the decision record" and the "lyx shuttle await" and "a later turn of the same session" bans.
  Each remaining want shrinks to the Go-filled value it carries, kept in the backticks the twill render puts around it so the template's own prose cannot satisfy it: "`lyx loom validate-plan --lens-review`", "`Step 2`", "`lyx loom validate-plan` per `Step 5`" (one want, since the bare "lyx loom validate-plan" and "Step 5" already sit in the plan template's own prose), the overview, plan-directory and lens-review paths, the lens line "Report defects only, never a passed check.", the dispute-evidence value and `planTwillSeverityRule`; the spawn-point lines stay.
  The "died before its Agent call" ban stays, as the anchor that the writer-fork part replaced the relaunch part; the order check keeps "## Step 2", "## Lens review by forks", "Spawn one fork, fork0" and "## Step 3" and drops "1. Read the rubric".
- `TestDiscussionTable`: drop "lyx board get" from the chair wants and the "Two narrow carve-outs" blank-line check (card 1 pins the support-log marker inside its section); keep the `## Step 5` blank-line check, the "twill" absence check and every path, strand and mode-rule want.
  Keep the `discussionTableSpecsDir` chair want card 1 adds.
  In the twill subtest drop `subagent_type: "fork"`, the "lyx shuttle await" and "a later turn of the same session" bans, and reduce the remaining wants to their Go-filled values, each kept in the backticks the twill render puts around it so the discussion template's own prose, which names "Step 5", "Step 6" and "lyx loom validate-discussion" itself, cannot satisfy it: the rubric path, "`lyx loom validate-discussion` per `Step 6`", "per `Step 5`", the two output paths in backticks, the rejection rule "Fix the record, or reject the finding with a one-line reason." and the severity rule "a severity (`BLOCKING`, `MEDIUM` or `LOW`)"; keep the "fork0" ban and the mode-rules-then-directive-then-Step-5 order checks.
- `TestReworkTable`: drop "lyx loom validate-plan --rework" and "numbered from 4 upward"; keep "`first_card: 4`" and every path want.
  The kept want pins the Go-filled first card, not the key: the stencil writes the key and the fill in one backtick span (`` `first_card: {{.first_card}}` ``), and that span is the shortest text that locates a one-digit fill, which a bare "4" does not.
- `TestParentReviewDeliveryPrompt_OneLineNamingReviewerSlugBrief`: drop "SendMessage", "lyx loom review delivered" and "--failed"; keep the reviewer, the slug and the brief path.
- `TestParentReviewBrief_NamesPathsAndSubmitLines`: drop "lyx board list" and "lyx board get"; keep the paths, the two slug-filled verdict lines and the none fills.
- `TestProducerSpecs_SkillsAndParentDirective` in `discussion_test.go`: shrink the `operatorBan` anchor from the full sentence to "never the way forward".
- `TestPrompt` in `internal/alignment`: the darn pr-gate row drops "Added after Discussion" and keeps "unrecorded parent correction" as the darn measure's anchor; the loom row keeps "a recorded decision the change breaks".
- `TestDarnSpec`: the first-spawn wants replace `` "`lyx board get '{\"slug\":\"fix-the-thing\"}'`" `` with the slug "fix-the-thing", and both `in.SpecsDir`-prefixed spec paths with `filepath.ToSlash(in.SpecsDir)` alone, the tails being prose.

**ImpactSummary:** test-only across `internal/loomengine`, `internal/alignment` and `internal/darnengine`; no helper signature or production code changes.
