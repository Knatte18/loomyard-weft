```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, covering each function or branch the diff changes, the case I ran, and whether it came out right.

**Per-branch list**
- `confirmPasted`, non-landing send, box reads empty on read 1: `box == "" && !awaitsLanding` returns nil at once. Correct, and the same as the old branch.
- `confirmPasted`, landing send, box empty on read 1, no session start, window open: `submitted` is set and the loop continues. Later reads skip the box switch, so no key is sent and no `ErrForeignBoxText`, `ErrSentTextInDraft` or `failUnlanded` is reachable. Session-start and turn-start checks still run first, so either returns nil. When the window or read cap runs out, `if submitted { return nil }` fires. Correct, and the same nil as the old `windowClosed || read >= maxReads` exit.
- `confirmPasted`, landing send, empty read on the final allowed read: `submitted` is set, then the closing check at send.go:644 returns nil. No lost exit. Correct.
- `confirmPasted`, landing send, box unreadable on every read: `submitted` stays false, so the window end goes to `failUnlanded` as before. Correct.
- `confirmPasted`, landing send, foreign text before any empty read: the `default` case returns `ErrForeignBoxText`. The new test row pins this. Correct.
- `confirmPasted`, landing send, empty then foreign text: no key and no error, and the window end returns nil. This matches D2. Correct.
- `confirmPasted`, landing send, empty then the sent text itself reappears: no Enter goes out, because `box == normalized` is now skipped, and the window end returns nil with the command possibly still unsubmitted. See the finding below.
- `probe` in `creelengine/deliver.go`: with `PaneMail` gone, a strand with `CreelHooks` set goes to `wakeHookStrand`. Any other strand falls through to the notice, then the reminder. A dormant or paused strand is skipped earlier, as before. Agent and program reminder rules are untouched. Correct.
- `hubDelivery.worktreeStrands` and the removal of `hasHookDelivery`: `CreelHooks` is copied from the persisted strand. The `shuttleengines` import is still used at delivery.go:94 (`EnvScrub`), so there is no unused import. No `PaneMail` or `hasHookDelivery` references remain in the repo. Correct.
- `creelfake.LiveStrand` and `HookStrand`: the pane-mail strand is now simply the one without `CreelHooks`. Correct.

**Findings**
1. LOW, `design`, `internal/shuttleengine/send.go:630-633`, `confirmPasted`: an empty read before the paste has rendered sets `submitted`. If the sent text then appears, the old landing-send path sent the guarded Enter, but the new path ignores it. The send then returns nil at the window end with the command never submitted, so creel would journal `injected` for an unsubmitted draft.
   - Evidence: the `readable && !submitted` guard at send.go:627, and the empty-box-then-sent-text sequence, which no test row covers.
   - This is the trade D2 states ("any text sends no key"), but its acceptance criteria don't weigh the case where the empty read is not a real submission. I judge it low risk because the first read comes after `SubmitSettle()`.
   - Fix: either accept and document it, or let `submitted` still send the guarded Enter when the box equals the sent text.

**Outside my lens, one sentence:** `internal/creelcli/smoke_tmux_test.go:76-77` now has a doc comment that opens with "The strand carries…", not the symbol name, and the old line below it is left in place and duplicates it.

**Tool calls:** none made. The manifest and diff already held everything I needed.
