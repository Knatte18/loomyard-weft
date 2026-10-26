```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that the round-3 changes agree with the text around them in `decision-record.md` and `support-log.md`.

1. **LOW**, in D9's rewritten InnerRun bullet (`decision-record.md:159-160`).
   Summary: the two new sentences read as if a driver-dead notice naming `lyx loom start` goes out at the cap, but the line before says only one notice goes out per state episode.
   Evidence: line 159 says the notice is sent "once per state episode of the child".
   Line 160 says "`respawnCap` changes only its text, which names `lyx loom start` once the cap's respawns have passed".
   Read together, a reader expects the cap notice to arrive.
   In fact the episode's one notice usually goes out at the first dead-driver probe, below the cap, with the plain text "driver strand is dead".
   The key is the same at and below the cap (`noticeKey(condition, state, episode.since)` in `notice.go`), so the text naming `lyx loom start` is reached only when no notice went out earlier in the episode.
   The new acceptance criterion at line 264 promises "one driver-dead notice in its state episode", which matches line 159 but leaves line 160's promise of the cap text unbacked.

The other changed text agrees with what it touches:
- **F2 (lines 149 and 263):** these fit the interrupted-landing bullet (line 155) and its criterion (line 261).
  That bullet removes the marker only when the status names Parent-Align, and a marker left standing with the status at Beam-Await posts nothing, which matches what line 149 intends.
  They also fit D7's bound at line 70.
- **F3 (lines 92-93):** these agree with each other and with `support-log.md`, which has no driver-dead or older-build twin.
- **F1 at line 257:** the removed clause has no twin left anywhere else.

Tool calls:
- grep of both target files for respawn, driver-dead, quiet notice, older build, `normalizeTask` and restart, to find sibling text outside the excerpts my context held.
- grep of `decision-record.md` for "notice" outside D9, to find twins in other decisions, the Rejected section and the criteria.
