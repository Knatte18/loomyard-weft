```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked the pairs of the record's own decisions that act on the same state, and checked the acceptance criteria and constraints against the decisions they touch.

**Findings**

1. **MEDIUM — D3, D5 and D4 against the existing done-run clear: a landed watch line that fails to post can be lost with no retry.**
   - D5 retries a failed post on later ticks, up to three attempts.
   - D4 says the memory "goes with the run's entry when the dobby clears the run".
   - The dobby's done-run clear (`internal/dobbyengine/doc.go`, "The done-run clear") removes a run whose batten reads done once its pair is gone. It waits only for an owed landing notice or board-tidy count, never for an owed watch line.
   - When `land_notice` is off, D3 has the watch report landed. A first failed post of that line is then followed by the clear in the same or the next tick, and the line is gone.
   - The record also never says the watch runs before `clearDoneRuns` in the tick, so even the first attempt can come after the clear.
   - Fix: count an owed watch line in the clear's condition, the way the landing notice is counted, and order the watch step before the clear.

2. **MEDIUM — D6 against the existing run notices: the gate-hold notice can be silently replaced.**
   - D6 tags the gate-hold notice "as the run notices are". That tag is `runNoticeTags(slug)`: Topic `runNoticeTopicPrefix+slug`, plus Run.
   - Under `PostTags.Topic`, a later driver-dead, api-error or usage-limit notice of the same run takes back an unread gate-hold notice, and the reverse also holds.
   - The gate-hold notice is the one line about a hang that D6 sends once per hold, so losing it brings back the 21-hour unseen hang of #216.
   - Fix: give the gate-hold notice its own topic, such as a gate-hold prefix plus the slug, or no topic.

3. **MEDIUM — D3 against D2 `stops`: the level posts progress lines during landing.**
   - Once the child reads done, D3 counts every reading as a stop, because the child is not running.
   - As batten then moves through Land, Worktree-Teardown and the rest, batten's row is part of the reading, so each move is a change "into a stop".
   - Under `stops` that posts a line per batten row. This is the progress feed D2 reserves for `rows`.
   - Fix: say whether a stop-to-stop change in which only a row moved posts under `stops`. The likely intent is that under `stops` only a change of a state part posts.

4. **LOW — D5 against D4: a pending line can be skipped when the reading moves on.**
   - D4 sends a line for any reading that differs from the memory, and D5 advances the memory only on a post or a drop.
   - If reading A's post is still retrying when the reading moves to B, the record does not say whether A is still sent.
   - Under `rows`, A can be skipped without notice, which defeats D5's stated aim of "every change, in order".

5. **LOW — D4 against D3: the memory is unclear for a landed line skipped because `land_notice` is on.**
   - D4 says the memory advances for a reading "told or skipped by the level".
   - The `land_notice` skip comes from D3, not from the level, so whether the memory advances on it is not stated.
   - If it does not advance, every tick re-judges the reading. That is harmless, but it stays undefined.

6. **LOW — D6 against D4: a gate hold's dedup memory has no stated home.**
   - D6 sends "once per hold, keyed by slot, PID and start", but names no place that keeps the keys.
   - D4's state record holds only the watch reading.
   - Kept in process memory alone, the keys are lost on a dobby re-exec and the same hold is noticed again. The run notices keep their keys in a marker file.

7. **LOW — D2 `off` against D6: the record does not say whether `off` covers the gate-hold notice.**
   - The two mechanisms are separate: one is a hub key, the other a per-task setting.
   - The record never says that `run_watch: off` leaves the gate-hold notice on.
   - The acceptance criteria do not pin it either.

**Pairs checked**

- D1/D4, the reading against its memory, including restart and the changed-status-file read: held.
- D1/D2 `rows`, whether every row change is seen: held. The dobby's file watch covers batten's status file and the pair; that was the extra read below.
- D2/D3 `stops`, a change back to running: held. During landing it is broken (finding 3).
- D3/D4, the `land_notice` skip and the memory: unclear (finding 5).
- D3/D5, the landed line going untagged so teardown keeps it: held for teardown, broken against the done-run clear (finding 1).
- D4/D5, memory advancing only on a post or a drop: held, with a gap when the reading moves on (finding 4).
- D5/D7, the orch dedup of a watch line followed by a batten notice: held.
- D6/D4, gate-hold dedup and restart: unspecified (finding 6).
- D6, existing run-notice topic: broken (finding 2).
- D6/D1, probe tick against every tick: held.
- D6/D2 `off`: unstated (finding 7).
- D3/D8, the landed line against the landing notice: held through D3's skip.
- D1/D8, the batten halt during the Land lock: held. The watch adds the line batten does not send.
- D7/D3, the orch role against the line prefix: held.
- Acceptance criteria against D1–D8: every id is covered. The criteria for D3 and D2 do not pin a row move during landing under `stops`, and the D6 criteria do not pin dedup across a restart.
- Constraints `PATTERN-run-fsm`, `config-layers`, `sole-parsers`, `daemon-wakeups` and `tolerant-records` against D1, D2, D4 and D6: held.

**Extra reads**

- `internal/dobbyengine/doc.go`, the done-run clear section, and `internal/beamcli/dobbywatch.go`: I needed the clear's conditions and the dobby's file-watch coverage, and my context held neither.
