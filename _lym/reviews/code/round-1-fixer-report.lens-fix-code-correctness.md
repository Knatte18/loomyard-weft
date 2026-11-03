```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: no findings that change a result.

I ran one concrete input through each changed branch:

- **F1, `ReviewRolePrefix` and `burlerDirName`:** the only callers are `p.rolePrefix(ReviewRolePrefix)`, `internal/loomcli/wiring.go`, `assist_test.go` and `engine_test.go`. All four use the new name, and the gate compiled.
- **F2, `unstartedAssist`:** with fan `nosuchfan` on an empty `lenses.Config`, `resolveRecordedFan` fails. `unstartedAssist` returns `reviewFilesIn(probeDirOf())` and the reason, so the seat's `instruction_1_path` and `instruction_2_path` are non-blank. `Table.Validate` then passes, and `Run` writes `not started: …`. The new `assist_test.go` row would fail with the defect in place, since blank paths make `Validate` return an error.
- **F3, `CapAssistTimeout`:** with `AssistReviewerTimeoutMin` 0, chair 60 min, start allowance 3 min and await cap 7 min, the room is 50 min. `want == 0` returns 50 min with no reason. The `room <= 0` branch still runs first, so a zero bound with no room still skips. Without the fix the row returned 0 and failed.
- **F4, `endingOf`:** a plain wait error, with `stopping` false, reads `wait error: wait failed`. A stop that ends the wait with an error reads `stopped`, because the fake `Stop` now releases with `stopWaitErr` after `markStopping`. Swapping the first two `endingOf` cases would fail the second row.
- **F5, wiring row:** with a 2m keep-warm cap, review timeout 10m and start allowance 185s, the bound is 295s. The default 250s cap gives 165s, which is what the failing run showed before the shortname and board-layer fixes. The row therefore fails if the cap is ignored.

One LOW observation, no change needed for the fix itself:
- **LOW:** `internal/loomcli/wiring_test.go` hardcodes `StartRetries: 1, StartupTimeoutS: 90`, copied from the shuttle template. A template change makes the row fail with a clear got/want message, but it is a pinned copy of a template value.

`internal/shedadapters/burlerreview.go:26` also holds a `"burler-review"` literal, `burlerReviewEngineLabel`. It is an engine label, not the role prefix, so I left it out of F1's scope.

Tool calls:
1. `grep -rn '"burler-review"' internal`, to check for other literals of the role name; my context only held greps limited to `burlerengine` and `loomcli`.
