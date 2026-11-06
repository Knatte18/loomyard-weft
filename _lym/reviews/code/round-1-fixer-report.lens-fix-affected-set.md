```yaml
lens: fix-affected-set
outcome: reported
```

Task: check through the affected-set lens that the F1 and F2 fixes cover every member of the sets their findings name.

No findings.

- **F1 (Post seam of each dobby):** the set is the `WaitReminders.Post` of the prime's dobby and of a beam's dobby. `TestHubDobbyDeps_WaitReminderPostReachesTheSlackRelaySignedAsTheDobby` has a row for each. It checks the sender (`<shortname>:DOBBY` for the prime, `<shortname>:task1:DOBBY` for the beam), the recipient `<shortname>:SLACK` and the body. A wrong recipient, wrong signing or swallowed error fails it, because `Post` is checked for nil and the posted-mail log is compared exactly.
- **F2 (two behaviours without a row):** both are covered. The `CancelMarker` row would post at minute 30 if `readWaits` stopped skipping the run. The `MoveRecords` row would count `awayReads > 0` if `tick` stopped gating on the anchor change.
- **Left unchecked:** none.

No further tool calls were made.
