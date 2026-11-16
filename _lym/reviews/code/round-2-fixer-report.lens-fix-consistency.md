```yaml
lens: fix-consistency
outcome: reported
```

The change set agrees with its surroundings. I found no findings.

- **F1 (verb constants):**
  - `RemoteControlBarVerb` and `RemoteControlOnVerb` now sit beside `RemoteControlLit` and `RemoteControlUnlit` in `bar.go`.
  - `panebin.go` composes its click and state lines from them.
  - `orchcli`'s `remoteControlBarVerb` and `remoteControlOnVerb` take their values from them.
  - The remaining `rc-on` and `rc-bar` literals are prose in doc comments and help text, or pinned expected strings in `panebin_test.go`.
  - Pinned expected strings in a test are meant to fail if the verb is renamed, so they are not duplicated declarations.
  - `RemoteControlOnVerb` and `RemoteControlLit` share the value `"rc-on"`.
  - Both are documented as separate things, and the value cannot drift in one without the other being noticed.
- **F2 (summary):** the edited bullets match the code. A click on a lit record refuses (`remotecontrol.go:145`), and `recordRemoteControl` writes nothing for an engine without remote control. The "What changes" RC-button bullet and the Tests list stay consistent with the new text.
- **F3 (test):**
  - The `wantLit` field comment says every row but the lit ones expects unlit. The new row with `wantLit: true` and the moved assertion both agree with it.
  - The assertion now runs on every row that reaches the success path. The `withdrawErr` row returns early, so it is unaffected.

Tool calls:
- `grep -rnE 'rc-on|rc-bar'` over the Go and Markdown files: checked for untouched twin literals that my context did not already show.
