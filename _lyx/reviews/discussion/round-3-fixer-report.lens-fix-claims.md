```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

1. MEDIUM. The changed text is D9's InnerRun bullet: "`respawnCap` changes only its text, which names `lyx loom start` once the cap's respawns have passed…"
   - **Summary:** this implies a notice naming `lyx loom start` reaches the orch at the cap. Within one running episode it never does when a notice below the cap has already gone out.
   - **Evidence, episode key:** `noticeCondition` (`internal/battenshed/notice.go:382-387`) gives the driver-dead finding the same stamp, `p.episode.since`, at and below the cap. `noticeStep` (`notice.go:277-282`) returns without sending when that key is already among the episode's keys.
   - **Evidence, episode lifetime:** `observeState` (`notice.go:248-251`) keeps the episode while the state stays running, and a respawned Beam-Await driver stays running. Respawns happen at `Call` (`innerrun.go:440-481`) and notices at the probe step (`innerrun.go:741`).
   - **Consequence:** the first death in an episode sends the plain "driver strand is dead" text with no way forward. A later cap-tripping death in the same episode sends nothing. So the cap stops respawning silently, and the only notice the operator got names no `lyx loom start`.
   - **Possible fix:** state that the cap's text goes out only when the cap is reached before any driver-dead notice of the episode. Alternatively, name `lyx loom start` as the way forward for every driver-dead notice at Beam-Await. That would be a build change, the reviewer's alternative.
   - **Other claims in the change set hold:** the new criterion's "one driver-dead notice in its state episode", F2's "`lyx beam start <slug>` passes since the status write lifted the hold", and F3's SetRunStatus and owner-setter claims.
     - D3 (`decision-record.md:49-60`) has no refusal on the dobby-stop marker. Its landing-hold refusal is lifted by the Beam-Await status write.
     - `Board.SetRunStatus` → `Store.SetStatus` (`board.go:185-193`, `store.go:601-614`) and `writeOwners` (`ownerset.go`) set fields directly. `checkProjectFields` is called only from `normalizeTask` (`task.go:289`).
     - Both replay paths call `normalizeTask` (`replay.go:309`, `replay.go:391`), so a replay of local commits touching the entry is refused.

Tool calls:
- Read `notice.go:240-258` plus a grep for the `observeState` callers, to learn whether a respawn starts a new notice episode. My context had only the key and delivery code.
- Read `innerrun.go:425-485` plus a grep for the `noticeStep` call, to learn where respawns sit relative to the notice step. My context had only the grep hits.
- Read `board.go:185-200` and `ownerset.go`'s `writeOwners` body, to confirm F3's claim that run-status and owner writes skip `normalizeTask`. My context had only the function signatures.
- Grepped `validateWrite` and the `checkProjectFields` callers, to confirm no other write path refuses a recipe. My context lacked that.
- Read D3 in `decision-record.md`, to confirm `lyx beam start` has no marker refusal, for F2's added claim. My context did not show D3.
