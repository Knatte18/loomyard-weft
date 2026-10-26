```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge each reviewer proposal the fixer carried out (F1's carried-over parts, F2, F3) by building a case where it is wrong.

Proposals checked:
- **F1** (fixed differently; I checked only the reviewer text that was carried over): "`respawnCap` only changes its text".
- **F2** (fixed as proposed): a failed dobby restart leaves the marker, and the notice names `lyx beam start <slug>`.
- **F3** (fixed as proposed): `SetRunStatus` and owner-setter writes "still succeed".

**MEDIUM: F1's carried-over claim that the cap changes only the text means the `lyx loom start` text can't reach the operator once a plain notice went first**
- Changed text: D9, "`respawnCap` changes only its text, which names `lyx loom start` once the cap's respawns have passed…".
- The case:
  1. A Beam-Await driver dies mid-wait.
  2. The wait loop's `noticeStep` (`innerrun.go:741`) finds it dead while `p.respawns < respawnCap`. It sends the plain "driver strand is dead" notice under key `noticeKey(noticeDriverDead, running, episode.since)`.
  3. The respawn happens only at the next pass entry (`innerrun.go:432-450`), and the child's state stays running, so `episode.since` does not move.
  4. When the cap later trips, the cap finding has the same key (`notice.go:383`, stamp `p.episode.since`). `noticeStep` returns early on a kept key (`notice.go:277-282`), so the cap text with `loomStartRemedy` is never sent in that episode.
- Result: after the cap, InnerRun spawns no further driver, and the operator holds only a notice that names no remedy.
- So "names `lyx loom start` once the cap's respawns have passed" states a behaviour the code does not hold whenever a below-cap notice went first. That is the common case for a driver that dies mid-wait.

**LOW: F3's new sentence contradicts the sentence before it**
- Changed text: D5 Upgrade: "so does the replay of its own local commits that touch it" followed by "Its `SetRunStatus` and owner-setter writes to the entry skip `normalizeTask` and still succeed, so an older machine can still overwrite the beam project's run status and owner".
- A run-status or owner write is itself a local commit that touches the entry.
- When origin has moved, its replay runs `normalizeTask(&incoming)` (`replay.go:391`) on origin's entry, which carries `recipe: pawl`, so the replay is refused.
- The overwrite therefore reaches origin only when its push needs no replay. Saying "still succeed" without that qualifier disagrees with the sentence before it.
- As a worst-case bound it is safe, but a reader will notice the conflict.

**F2: no finding.**
- `lyx beam start` passes once the Beam-Await status write has lifted the hold (D3's landing-hold refusal is the only one that hold triggers).
- If the restart failed because an anchor move is standing, `lyx beam start` refuses naming the move's way forward, which still makes progress.
- `EnsureLive` leaves a live dobby alone, so a stale marker at the next failed accept is harmless.
- A passing accept removes the marker, and the interrupted-landing path at Parent-Align removes it too.

Further tool calls:
1. Grepped `validateWrite`, the dobby-stop marker in code, and `lyx beam start` mentions in the record, to see whether anything besides Beam-Await reads the marker or refuses a start. My context lacked the D1/D3 text and the marker readers.
2. Read D1–D3 (record lines 30–60) for `lyx beam start`'s refusals, which my context had not shown.
3. Read `innerrun.go:420-485` and the `noticeStep` call site, to see whether a respawn comes before or after the notice in a probe. My context had only the grep hits.
