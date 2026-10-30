```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F3, F4, F5, F6 and the test additions), through the fix-proposal lens.

No findings.

- **F3 (status-write reset in `RespawnCapReached`, memory error reads as not capped):** I could not build a case where the reset is wrong.
  - Equal times read as capped, and a later status write reads as not capped. That is the comparison the row's own pass uses (`baseline.After(p.respawnStatusTime)`).
  - The mtime passed in is the one `readNoticeChild` already stats for the status file, so there is no second stat to race.
  - An unreadable memory record now loses only the cap wording of a driver-dead notice. The notice itself still goes, which is the harm the finding named.
  - `RespawnCapReached` has no other callers. Its signature change touched only `readNoticeChild` and the existing battenshed test.
- **F4 (one cap constant):** `RespawnCap = respawnCap` gives one value for the reading and the text. `ChildReading.RespawnCap` replaced the bool, and the only users I could see in context were the dobbyengine text, its test row and `readNoticeChild`, all updated. Zero meaning "not capped" cannot collide with a real cap of 3.
- **F5 (`runShedDuration` falls back on a non-positive count):** the recipe validator already rejects negatives and defaults a zero probe. A hand-edited zero quiet value now keeps 30 minutes, so the quiet notice can no longer be turned off from the recipe. The review offered that as one of its two options, and the fix's doc comment states the behaviour.
- **F6 (delete `resetText` and `resetStamp` from battenshed):** the module-wide build was refused, so I rely on the package builds, which passed (the `battenshed` build after the edit, and later `battenshed`, `dobbyengine`, `battencli` and `beamcli` together). dobbyengine keeps its own copies.
- **Test additions (F1, F2, F7, F8, F9):** the mutation runs reported in context show F1, F2, F7 and F9 fail with the defect in place.

Tool calls beyond the context: none made.
