no findings

Ran each changed branch by hand: `isTopBoxRule` (F1), `noticeStartPromptHeld` (F2), the interactive row (F3) and the reason literals (F4).
Each added test fails with its defect in place; the gate passed after the final edit.

Tool calls:
1. grep for the notice text, `noticeStartPromptHeld` and `cutLabelRuleEnd` across the repo; the context held no repo-wide search.
