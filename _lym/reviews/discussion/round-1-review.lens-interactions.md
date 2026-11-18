```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked pairs of decisions in the decision record that act on the same state, then checked the acceptance criteria and constraints against those pairs.

**Pairs checked**
- D4 launched-account record × D7 in-run relaunch: broken (F1)
- D4 "a reed relaunch replays the stored command" × D7 in-run relaunch: broken (F1)
- D5 status line in `--settings` × D7 in-run relaunch: broken (F1)
- D5 `blocked_until` on the launched account × D7 switch keyed on the launched account: broken (F1)
- D4 pin skip on `blocked_until` × D7 "with none eligible, changes nothing": broken (F2)
- D4 pin skip × D3 "no pick for a pinned worktree" (a pinned worktree with no entry): broken (F2)
- D2 cache wipe × D4/D7 pinned switch: broken, same as F2
- D1 windows-only judging (no default threshold) × D5 `blocked_until` "no headroom in any window" × D3 eligibility: broken (F3)
- D5 `blocked_until` × D3 step 3 "latest blocking reset": broken (F3)
- D3 counter normalisation (`counts`/`weights`/`listed`) × D7 switch increment: broken (F4)
- D1 retired account × D2/D4 an existing entry naming it: broken (F5)
- D1 retired account × D7 `set` × D1 pin refusal: held (set and pin refuse; the gap in F5 is the entry case)
- D1 "fewer than two positive-weight accounts" × D7 switch and set × pins: held
- D2 name-keyed entry × D8 removed account drops from the known directories: held
- D3 prune × D9 prime entry from set or switch: held
- D6 newest-copy rule × D8 `LocateSessions` most-recent dedup: held
- D6 copy before the live-holder test × D7 set leaving the strand live on the old directory: held (registries are read as one set and refuse; the copy is redone at the next resume)
- D6 × D7 two sessions switching at once (one switch, both resume): held
- D4 scrub × D8 standalone readers (process `CLAUDE_CONFIG_DIR` plus `~/.claude`): held
- D4 effective directory × D8 known directories including the `env` value: held
- D5 status line only for a listed launched account × D4 unlisted directory: held
- D7 set × D4 pin "not out": held
- D7 orch restart × D6 copy × D9 beams pin: held (it returns to the pin only at the next launch, which D7 states)
- D1 tracked-layer refusal (launch fails) × D8 "fails to load is logged; readers go on": held (refusal on launch, degrading on read are separate roles)
- Acceptance criteria: the D4 AC "pinned account with a future `blocked_until` yields to the entry" and the D7 pinned AC cover only the case where an entry exists (F2); the D3 AC "every candidate short" relies on the undefined blocking reset of a `blocked_until`-only account (F3); the others are consistent with their decisions.
- Constraints: PATTERN-shuttle-provider-seam, lym-files, tolerant-records and cwd-resolution are consistent with D2, D4, D5 and D7 as written.

**Findings**

F1 — BLOCKING, design. Location: `_lym/discussion/decision-record.md` D7 (in-run relaunch), with D4 and D5.
Summary: D7's relaunch inside the same run does not say that it rebuilds the launch through the engine (`Prepare`). If it doesn't, three things stay on the old account A:
- the run directory's launched-account record (D4);
- the `--settings` status line naming A (D5);
- reed's stored command, which D4 says "a reed relaunch replays".

The session now running on B then has its status line write B's figures into A's usage file. Its next limit hit records `blocked_until` on A (D5 "on its launched account"). D7 then finds the worktree "resolves to another account with headroom" (B, unblocked) and relaunches under B again, in a loop, while D8's live readers read A's directory.
Evidence: decision-record.md lines 110–111 (only `Prepare`/`OpenLaunch` write the record), 116 (reed replay keeps the launched account), 126–127 and 137 (status line and `blocked_until` keyed to the launched account), 172–174 and 180 (switch keyed to the launched account, relaunch). `internal/shuttleengine/run.go` around line 677 stores `ResumeCmd` from the launch, and `attach.go:413` says "Attach never relaunches", so no existing in-run path rebuilds a launch.
Fix: state that the in-run relaunch builds a fresh resume launch through the engine, rewriting the launched-account record, the settings and the stored command, and add that to the D7 AC.

F2 — MEDIUM, design. Location: D4 resolution order with D7's "with none eligible, changes nothing" and D3's "no pick for a pinned worktree".
Summary: a pinned worktree whose pinned account has a future `blocked_until` skips the pin "in favour of the entry". A pinned worktree has no entry (D3 never picks it; a wiped D2 file loses one). When D7 finds no eligible account, or before it runs, D4 therefore resolves to "none", and every launch exports nothing and runs under the `env` value or `~/.claude`. That is an arbitrary, possibly short or unlisted account in place of the pin. It contradicts the board's "waited out only when no account has headroom" and the hub pin's "every worktree runs under it".
Evidence: decision-record.md lines 101–102, 79, 176, 178; AC line 276 covers only "yields to the entry".
Fix: when the pin is out and no entry exists, keep the pin (wait it out); add that case to the D4 and D7 ACs.

F3 — MEDIUM, consistency. Location: D3 steps 2–3 against D1 `windows` and D5 `blocked_until`.
Summary: D3 calls a candidate eligible when every window it lists has headroom. D1 allows an account to list no windows ("judged only by the windows it lists"), so a `blocked_until` account with no listed windows is vacuously eligible. D7 can then switch onto an account that is out, despite D5's "has none in any window". D3 step 3's "latest blocking reset" is also undefined for an account blocked only by `blocked_until`.
Evidence: decision-record.md lines 44–45, 86–87, 140.
Fix: make a future `blocked_until` disqualify an account regardless of its windows, and count it as a blocking reset in step 3.

F4 — LOW, consistency. Location: D7's switch with D3's counter normalisation.
Summary: D7 runs D3's steps 1 and 2, then "writes … entry and increments its count". It does not say it rewrites `weights` and `listed`. A switch to an account added since the last pick therefore has its count reset again by the next pick's "not listed at the last pick" rule, so the switch's increment is lost.
Evidence: decision-record.md lines 65, 88–91, 175.
Fix: say the switch writes the counter file as D3's step 4 does: the normalised counts, the weights and the listing.

F5 — LOW, consistency. Location: D1 retired accounts with D2/D4 entries.
Summary: D1 says a weight-0 account is "never picked or switched to", and D1 refuses a pin naming one so the launch fails. An existing entry naming a now-retired account is still listed, though, so D2 does not drop it and D4 keeps launching that worktree under the retired account. Retiring an account therefore moves no worktree off it, and the record states neither this nor the asymmetry with the pin refusal.
Evidence: decision-record.md lines 43, 52, 67, 101.
Fix: state whether an entry naming a retired account resolves (and add an AC either way).

Tool calls: none.
