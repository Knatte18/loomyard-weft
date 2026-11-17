```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 fix change set.

No findings.

- The overview's `*` matchers match `toolMatcherAll` in `command.go:280`, `:288`.
- The overview's `request_user_input` and `request_user_input_async` deny matches `questionToolPattern` in `guards.go:84`.
- `CountLinesUpTo` now returns at most `limit`. Its other caller, `claudeengine/batchingnudge.go:497-501`, only tests `lines < SliceLines`, so the clamp does not change that result.
- `localPath` is the only `file://` parser in `shuttleengine`; no twin was left unchanged.
- I did not confirm that `questionDeny` is set only for autonomous launches, so the overview's word "autonomous" rests on the reviewer's reading.

Further calls:
1. `grep` over `claudeengine/batchingnudge.go`, `shuttleengine` and `codexengine`: checked the `CountLinesUpTo` caller, the `file://` and `url.Parse` sites, and the deny and matcher constants, which my context lacked.
2. `sed -n 268,284p` of `command.go`: read where `questionDeny` gates the `*` PreToolUse entry, which my context lacked.
