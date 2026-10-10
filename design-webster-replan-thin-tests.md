# Thin test rows for the fabric classifier, rebaseline and shell wait

`webster-replan-thin-tests` · Note · enhancement, webster, fabric, shuttle

## Seen

`webster-replan-state` landed (5a8be0274) with findings `card14-classifier-rows-miss-glob-and-tilde` and `thin-test-rows-cards-13-11-17-21` (MEDIUM) open: the classifier tables lack glob and tilde cases, and the rebaseline and shell-wait tables are thin.

## Idea

Add those rows.