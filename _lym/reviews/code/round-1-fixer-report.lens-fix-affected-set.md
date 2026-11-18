```yaml
lens: fix-affected-set
outcome: reported
```

Checked the four set-premise findings; no findings.

- **F4 (every `decisionDeps` constructor):** the verb's preRun builder at `internal/loomcli/decision.go:1538` and the test fake at `decision_test.go:269` both set `judgeSessionOpen`. The third constructor, `decision.go:1908`, sets only `recordPath` and `programName` and leaves it nil. I did not read the verb that constructor serves. By its fields it looks like `decision store`, which never reaches the `--amends-plan` and PATTERN-change check in `decisionVerb` (`decision.go:994`, the only call to `deps.judgeSessionOpen()`), so a nil-func panic is not reachable from it.
- **F1 (leftover mentions of the removed refusal):** the phrases "covers no line" and "every line is keyed" survive only in `_lym/reviews/code/round-1-*` files. No code, doc comment, stencil or `seats.go` text still states the refusal.
- **F3 (callers of the changed signature):** `appendedDecisionRecord` has one caller, `judgesnapshot.go:249`, already updated to pass `records`. `recordsCheckoutPath` is still used by `TakeJudgeSnapshot` at `judgesnapshot.go:172`, so it is not dead.
- **F4 (other verb writes outside the exemption):** `weaverengine.RecordDelivery`, the other side effect of the verb that could write to disk, only mutates state under `ScratchDir(anchorRoot)`. By its name that is scratch space, not the records checkout, so it is outside the snapshot's records comparison. I did not read the full function body.

Tool calls (one combined Bash call; the other five were unused):
- One grep batch for the `decisionDeps{` constructors, `judgeSessionOpen` uses, leftover refusal text, `appendedDecisionRecord` and `recordsCheckoutPath` callers, and `RecordDelivery`'s write path. My context lacked these repo-wide sites.
