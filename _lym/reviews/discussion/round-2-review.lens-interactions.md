```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, ordering or line.

**Pairs checked**
- D1 (tick order) × D3 (copy of the land_notice reading): held. `landNotices` runs before the watch step (`loop.go:719`). `readLandNoticeRequests` skips runs past Land (`landnotice.go:75`), so the watch's last copy taken ahead of Land matches the reading the landing notice decides with.
- D1 (`listedRuns`) × D4 (separate watch map) × the done-run clear: held. `recordOwed` touches only `s.Runs` (`passes.go:333-337`), and the clear runs after the watch (`loop.go:729`).
- D2 (never-read level) × D4 (entry for every level; first sight sends one line): held.
- D2 `off`/`rows` × D3 skip × D4 (skipped reading still becomes memory): held.
- D2 stops rule × D1 episode stamp (layer switches when batten leaves running): held. Every switch comes with a batten state change.
- D3 stop predicate × D3 "the watch posts no line" for a cancel that removes its records: **broken** (F-A).
- D3 skip × D4 upgrade with no memory or copied reading: held, and the record states it.
- D4 owed lines × D5 retry and drop × the clear's hold: held.
- D1 scheduling bound × D5 one-hour cap: **broken** (F-B).
- D1 scheduling bound × D7 orch-role rule: **broken** (F-C).
- D4 (gate keys in the watch entry) × D5 (delivery rules) × D6 (one notice per hold): **broken** (F-D).
- D6 × D2 `off`: held. D6 × D4 restart dedup: held.
- D2 × Constraint PATTERN-config-layers, D5/D6 × PATTERN-daemon-wakeups, D4 × PATTERN-tolerant-records: held.
- Acceptance bullets against D1–D8: consistent, except those touched by F-A and F-D.

**Findings**

F-A — MEDIUM, consistency — `_lym/discussion/decision-record.md:88-90` (D3 cancel paragraph), with D3:76 and the Goal at :9.
- Summary: D3 counts `cancelling` as a stop. A cancel writes its intent into the batten record before ending anything. So a `--remove` cancel derives Cancelling while it works, and a tick that sees it posts a `[run watch] … cancelling` line. That contradicts D3's "the watch posts no line for it".
- A cancel that keeps its records posts two watch lines, cancelling then cancelled. The Goal and the acceptance criteria name only the cancelled line.
- Evidence: `internal/battencli/cancelcmd.go:315` writes the intent with `Remove`; `internal/runfsm/state.go:80-82` derives Cancelling; dobby ticks on status-file events (`loop.go` `sleep`).
- Fix: either drop `cancelling` from D3's stop set, or state that a cancel posts a cancelling line, and align the Goal and acceptance criteria with that.

F-B — LOW, design — `_lym/discussion/decision-record.md:124` (D5 hour cap), with D1:45-46.
- Summary: D1 runs the watch only on scheduling ticks, so an adopt hold, anchor move or catch-up can stop posting for a long time. D5 measures the one-hour cap in awake time whatever the cause.
- Result: a line owed before such a hold can be dropped on the first tick after it ends, with no post ever tried. That defeats D5's three-attempt rule and D1's promise that a hold only delays lines.
- Evidence: `internal/dobbyengine/loop.go:708-729`.
- Fix: count the cap only over watch-step ticks, or exclude held time.

F-C — LOW, consistency — `_lym/discussion/decision-record.md:166` (D7), with D1:45-46.
- Summary: D7 tells the orch that a *later* `[batten notice]` for the same stop opens no second fork. But D1's bound lets the watch line arrive *after* batten's notice, behind a catch-up after sleep or an adopt hold. The orch role says nothing about a watch line that follows the detailed notice, so the orch may open a second fork.
- Evidence: `loop.go:701-708`; `contracts/orchrole/orch-role.md` Notices section.
- Fix: word D7 for either order: one stop, one fork, whichever line comes first.

F-D — LOW, design — `_lym/discussion/decision-record.md:145-150` (D6), with D4:103-104 and D5:121-124.
- Summary: D4 stores gate-hold keys and owed watch lines in the same entry, but D5's delivery rules (wait with no strand, probe spacing, three attempts, one-hour cap) cover watch lines only. D6 does not say what a failed gate-hold post does.
- If the key is recorded on the attempt, the notice is lost. If it is recorded only on success, the post is retried every probe tick with no cap.
- Evidence: `internal/dobbyengine/landnotice.go:158-159` is the existing uncapped-retry precedent; `beamcli/dobbydeps.go:199-202` runs posts through `requirePosted`.
- Fix: state in D6 when the key is recorded and which retry and drop rule a failed gate-hold post follows.

Tool calls:
1. grep of the cancel code for where a cancel intent is written to the batten record: my context held `clearRunPhase` and the `Cancelling` derivation but not the intent write, which F-A rests on.
