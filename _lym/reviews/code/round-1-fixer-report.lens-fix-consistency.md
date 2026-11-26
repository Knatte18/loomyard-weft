```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F3 change set against its surroundings.

no findings

The new `docs/overview.md` wording agrees with `internal/slackengine/doc.go`, `docs/slack-app.md` (`Relay commands`) and the `handle` doc comment in `inbound.go`. A search for the old bare-command wording turned up no stale twin. The new rows (`APresenceCommandWithMoreWordsIsForwarded` and `MadeByThisMachinesBotPostingWithItsUserIDAndNoBotID`) agree with their tables and with the documented forwarding and own-post behaviour.

Tool calls:
- Searched docs and the slack packages for other wording that names the bare commands: my context held only the overview's slack entry and some code, so it couldn't show whether a twin existed elsewhere.
- Read the relay-command text in `internal/slackengine/doc.go` and `docs/slack-app.md`: neither passage was in my context.
