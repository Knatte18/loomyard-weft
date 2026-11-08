```yaml
lens: code-affected-set
outcome: reported
```

Task: find what the diff's changes force outside the lines it touches (lens code-affected-set).

**Per change: the set found, and whether covered**

1. **Merged Stop command now runs the turn hook before creel's delivery.**
   - Covered: `claudeengine/doc.go`, the `buildSettings` doc comment, `creelStopHookCommand`'s doc, `settings_test.go` (`wantMergedStop`), `hookcmd_integration_test.go`.
   - Covered: `creelOpenStopHookCommand` has no turn hook, and `claudeengine/command_test.go:656` compares it against the function itself.
   - Not covered: `shuttleengine/spec.go:151-152` (see F1).
   - Not covered, and not a defect: `codexengine`. Its Stop command only records, and it has no turn hook (grep found no `TurnHookCommand` handling there).
   - Checked, no issue: `slackcli/hook.go` `replyHook` reads the presence and posts the reply. It does not depend on the order against creel's delivery. `orchcli/start.go:188` only names the Slack hook as the turn hook.
2. **`afterEmptyPoll`, with no wait after a poll that waited out the floor.**
   - Covered: the `backoff.go` header, the `read` doc in `outbound.go`, the `RunRelay` text in `slackengine/doc.go`.
   - Not covered: other docs. No other doc in the repo describes the 5 to 60 s doubling (my grep found none).
3. **Presence notice skips the orch the operator's message just woke.**
   - Covered: the `noticePresence` and `noticePresenceChange` docs, `slackengine/doc.go`.
   - Not covered, and not a defect: `contracts/orchrole/role-slack.md:8` still says the notice "tells you the new presence". It stays true for the other orchs. For the skipped orch the forwarded Slack mail carries the same fact.
   - Not covered, and not a defect: `slackcli/doc.go:11` says only that a relay tells the hub's orchs, which stays true.
   - The `noticePresence` signature has only the one caller, `relay.go:696`, and it is updated.
4. **PATTERN.md and `pattern/`.** The diff touches neither. No `Pattern-Departure:` trailer appears in the commit messages, and none is needed. No declared PATTERN changes were visible to me.

**Findings**

- **F1 — LOW, consistency.** Location: `internal/shuttleengine/spec.go:151-152`. The `TurnHookCommand` field doc says the hook also runs "at each turn end of the session's main thread that hands the session no creel mail". That is now false, because the merged Stop command runs the turn hook whether or not mail is handed out. Evidence: `settings.go:134-139` puts `turnHookStep` before `creelStopDelivery`, and the `claudeengine/doc.go` change states it. Fix: reword line 152 to say the hook runs at each turn end of the main thread, whether or not creel mail is handed to the session.

**Tool calls**
1. Grep for docs and strings describing the old Stop order, the backoff and the presence notice, across `.go`, `.md`, `.tmpl` and `.yaml` — to find the affected documentation.
2. Grep for turn hook and `hook-read claude` references outside `settings.go` and `doc.go` — my context lacked them.
3. Read `slackcli/hook.go` and `role-slack.md` — to see how the Slack hook and the orch role text depend on the Stop order and the presence notice.
4. Grep for turn hook in `codexengine` and `shuttleengine`, and a PATTERN diff check — to see whether a sibling engine or the spec doc was affected.
5. Read `spec.go:148-159` — to confirm the `TurnHookCommand` doc text.
