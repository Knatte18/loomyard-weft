---
round: 1
focus:
  - "Answer key is the board entry alone (empty Scope/Decisions): check `code_review_ask_parent` and `drift_judge_ask_parent` each exist, are settable in `loom.yaml`/`drift.yaml` and in a task's `run_settings`, and each has a timeout key as `discussion_review_parent_timeout_min` has."
  - "`internal/shedadapters/parentseat.go` and `internal/loomengine/review.go`: does each code review round mail a `[code-review request]` naming the review and fixer report, and wait for `lym loom parent-review answer` (satisfied or findings file) before the Bouncer judges? Check the timeout path and the cap in `CapParentReviewWait`."
  - "`internal/driftengine/judgeparent.go` and `verify.go`: does the judge mail its proposed rulings (every cell or a summary by case with flagged cells named) and wait for agreement or changed rulings before writing the judgement, in both Drift-Verify and Drift-Recheck? Check the held-ask path (`heldJudgeAsk`, `agreementGate`) and the timeout and failure paths for broken invariants."
  - "`internal/loomcli/parentseatwiring.go`: do the new keys reach the right segments (Code-Burler-Review/Fix, Drift-Verify/Recheck) and not the discussion or PR seats? Survivors at lines 62-64 and 76-78 drop statements with no test killing them; check whether a test could assert the wiring."
  - "Scope: the diff touches `contracts/orchrole/orch-role.md`, `docs/overview.md` and doc.go files; check each change is licensed by the entry. Routing agent questions to the parent and parent notification on named rows are out of scope."
  - "Mutation survivors in `judgeparent.go` lines 71, 138, 144 and 162 (gate entry constant, `postJudgeAsk` and `agreement` conditions): decide whether a test could reasonably assert the diff's own behavior there."
  - "Test economy: `TestJudgeAskLine`, `TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments` and `TestParentAskLine` are listed as contained; compare their assertions against the named covering tests before filing."
---

Seed focus only; no prior round exists.
