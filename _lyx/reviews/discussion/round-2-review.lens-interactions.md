```yaml
lens: interactions
outcome: reported
```

**Pairs checked**

- D3 × D4 × D7 (owner takeover judged by a full pass's reconcile, owed-full-pass rule, failing-pass notice): **broken**.
- D4 spawn-hold yield × D4 memory record (owner triggers persist): **broken**.
- D4 "one check per Call" × D4 memory record (owner pause-lift/retry triggers): **broken** (which check runs is unstated).
- D4 new respawn count × D9 api-error episode counters × D4 inventory: **broken** (no stated home across passes).
- D2 (`lyx orch start` calls `lyx beam start` first) × `lyx orch start`'s own owned-elsewhere pause × D4 passes of covered running runs × D13 backoff: **broken** (symptom of finding 1).
- D12 lift pass × D4 pass-in-flight via held driver lock: held.
- D12 no-Land-under-pause × D4 continue pass already in Land's lock wait: held as worded ("enters").
- D12 × D13 (covered running runs count as in flight): held.
- D5 draft-on-cancel/claimClear × D4 battenAlive change: held (running → kept; halted → draft; owned-elsewhere `fabric remove` is `Local`, no board write).
- D5 start guard × D4 hand pass starting the parent's dobby: held.
- D5 migration key × D5 creating-write default: held.
- D5 entry-change retry × D7 refusal notice clearing: held.
- D7 failing-pass notice key × D4 three-failure rule: held.
- D8 row notices held for the stop chain × D9 kept Land/Teardown notices: held.
- D8 arbiter/tackler sessions × D4 in-pass wait bounds × D12: held.
- D4 `Yield` × D8 stop chain (yield is no stop): held.
- D9 driver-dead × D4 respawn cap: held.
- D1 `RoleDobby` under beam slug × D4 `battenAlive` "live dobby drives it": held.
- D1 reed-down spares dobby × D10 tidy agent in the beam's reed session: held (killed tidy bounded by `board_tidy_timeout_min`).
- D1 owned-elsewhere exit × D2 `lyx beam start` with another prime owner: held.
- D4 hand pass is a full pass × D4 owed-full-pass kept in dobby's record: held (dobby cannot see the hand pass's exit, so it may run one extra full pass — harmless).
- D4 `--continue-pass` gate × D4 re-exec keeps owed full pass: held.
- D4 status-write trigger (mtime later than last pass exit) × D6/D12 hand pass resuming a halted run: held.
- D13 in-flight definition × D5 `max_runs` counting halted: held.
- D6 × D4 (dobby never resumes): held.
- D8 migration × Constraint PATTERN-config-layers: held.
- Acceptance "a pass under a standing owner yields at once" × spawn-hold gap: broken (covered by finding 2).

**Findings**

1. **BLOCKING, design** — Location: D3, D4 (owed full pass, retries), D7.
   Summary: A run another machine took over is driven forever and InnerRun's owned-elsewhere handling never runs.
   The full pass's origin reconcile refuses an owned-elsewhere entry (`resumeRefusal` → `ownedElsewhereRefusal`, `internal/battencli/resumereconcile.go:80`), a plain non-zero exit, not the `--continue-pass` refusal code. D4 clears an owed full pass "only when a full pass of that run exits 0" and retries a failing pass every probe tick, with "the retries go on" after three; so the run gets a refused full pass every `notice_probe_s` forever, D7 sends a failing-pass notice instead of today's `ownedElsewhere` pause + parent-lock release + owner notice with the adopt line (`internal/battenshed/innerrun_owner.go:105-121`), and D13 counts it as in flight so dobby never backs off. D3's "it drives only runs this machine owns" has no mechanism for dobby to learn of the takeover and stop; the lock is only released if `lyx orch start`'s `pauseOwnedRuns` (`internal/orchcli/start.go:285-300`) happens to run.

2. **MEDIUM, design** — Location: D4 (spawn hold yields) × D4 memory record.
   Summary: A persisted standing owner makes every pass yield with no owner re-check, so a run whose ownership came back stays held.
   Today `holdSpawnWhileCovered` itself re-runs the owner check once per notice probe while `ownedBy` stands (`internal/battenshed/innerrun_pause.go:47-69`). D4 makes the hold return `Yield` at once, so that in-hold re-check never runs; the persisted triggers (`foldOwnerTrigger`/`ownerStep`) run only inside `check()`, which the spawn path never reaches (`internal/battenshed/innerrun.go:414-417`, `700-710`). A run whose `PauseOwned` failed (so no pause covers it) and was adopted back yields on the persisted `ownedBy` every pass and never clears it.

3. **MEDIUM, design** — Location: D4 "one check of its child per Call".
   Summary: The record does not say a pass's one check includes the owner fold and step; today's first round of a Call does not, so the persisted pause-lift and retry triggers may never fire.
   `wait()`'s first round runs only `coverage`, `noticeStep`, `runArm` (`internal/battenshed/innerrun.go:669-678`); `foldOwnerTrigger` and `ownerStep` run only in `check()` after the first `Sleep` (`innerrun.go:700-710`). D12's lift pass relies on the persisted `ownerCovered` → released trigger, which fires only if the pass's one check is a `check()` round.

4. **MEDIUM, design** — Location: D4 memory-record inventory × D4 respawn cap × D9 api-error.
   Summary: New cross-pass state introduced by D4/D9 — the respawn count and its last-respawn status-write baseline, the api-error revive-sent mark per episode, and the count of revived episodes in a row — is placed nowhere, so the caps can reset every pass.
   D4's inventory lists only today's producer and `childWait` fields; "Every other rule stated per batten process today becomes per episode" does not cover state that does not exist today. Since each pass is a new process, without a stated home the respawn cap (≤3) and D9's "third revived episode in a row" fall back to per-process memory and never trip.

No tool calls were made; all evidence was in the inherited context.
