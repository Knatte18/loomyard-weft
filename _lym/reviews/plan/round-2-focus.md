---
round: 2
focus:
  - "Cards that plan tests for the adopt worker and caller follow: does any planned test function pin behaviour that another planned test of the same surface already covers, or test an unexported helper directly instead of through its surface?"
  - "The adopt step order and the worker's merge of per-step outcome parts: is the rule for deriving a part and merging parts (scalars, per-slug fields, lists) stated, and does it keep every field the cards' own test rows expect kept?"
  - "The caller's follow of a hidden worker: for every record state reachable while following, including a record rewritten by another adopt and a worker that has exited but is not yet reaped, does the plan name the input and the test that reach each branch, and does the follow end?"
  - "The adopt command's `Long` text and the CLI-tree walk in `cmd/lym/clitree_test.go`: after every card that edits `Long`, does the text stay within the walk's byte cap?"
  - "The text set the green end and the hold retirement falsify (error texts, comments, docs/overview.md adopt entry, fabricengine scratch-directory docs): does some card reword each text that still states the old behaviour?"
---

The focus lists where to look; it does not cap severity or prejudge a verdict.
