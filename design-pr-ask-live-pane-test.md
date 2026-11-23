# No tmux-tier test sends parent-review ask from a live non-orch pane

`pr-ask-live-pane-test` · Note · bug, landing, creel

## Seen

`failed-ask-hides-satisfied` (issue 213) left out its item 3: no test runs `lym loom parent-review ask` from a live non-orch reed strand pane.
That is the path that failed twice on 2026-11-23 in the mo hub (creel refused the LOOM signature), and the fix (708cdadc5) is pinned only below the tmux tier.

## Idea

A tmux-tier test: a built lym binary in a reed strand pane, a hub with a pull request record under autonomy, and a live orch strand that receives the post; assert the ask is delivered and the parent's answer lets `pr-continue` end the review.
