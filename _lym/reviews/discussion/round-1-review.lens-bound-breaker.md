```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guard and claim about existing code in the record.

**Claims attacked**

- D2 bound, "only a raddle run turns a Sketch into tasks": **broken.** An operator can take the `sketch` label off the note, then promote it with any recipe.
- D2, "the board admits `sketch` on a task only while its recipe is `raddle`": held. The rule is new, and no existing write path conflicts with it.
- D7 bound, "both verbs refuse outside Raddle-Write; approval without operator input is prevented only by the stencil": held as stated. One more bypass is unstated: the agent can turn the auto key on in the pair's local config layer.
- D7, "Raddle-File files only a proposal whose hash matches the recorded approval when the key is off": **broken.** The record never says when the key is read, and the key can change inside the pair after approval.
- D7, "the cold proposal is stored once per run": held. Its waiting notice also goes out only once per run, so a revised proposal after a D8 return asks for a new approval with no new notice.
- D8, "an entry gone from the board has landed": **broken.** `lym board remove` deletes an entry that never landed.
- D8, "a new task or project absent from `board_checked` sharing a module label fails": **partly broken.** An existing task that gains the Sketch's module label after the check passes.
- D9 bound, "Raddle-File writes the board as Go code with no strand name, so the agent's own `lym board` calls are still refused": held. Only `boardcli` supplies a strand name, and `checkTaskCreation` judges only that.
- D9, "the run lock admits the done status": held. `scopeChanged` ignores Status (`internal/boardengine/runlock.go:185-217`). The record does not cite this as evidence.
- D9/D10, "later status writes skip a done entry": held. `SetRunStatus` returns without writing for a done entry (`internal/boardengine/board.go:203`), and `MakeSketch` does the same (`sketch.go:77`). Not cited either.
- D9, "every step is skipped when its result is already there, so a retry finishes the missed steps": **at risk.** The record names no done-marker for step 3, and D8's slug check runs inside that write.
- D9/D10, "Land prunes the done Sketch": **partly broken.** A done entry with `notify_on_land` set is not pruned, and its landing notice fires mid-run.
- D11 claim about `NotifyOrchAt`: held (`internal/orchcli/notify.go:68-70`).
- Open risk 3 (Codex cache-write split): the repository already answers it, so the risk is a deferred fact.

**Findings**

1. MEDIUM · design · D2 Bound
   - Summary: the stated bound does not hold. The guard keys on the `sketch` label, and a plain note write can drop that label.
   - Counterexample: the operator upserts the Sketch note without `sketch` (a note write, which no rule refuses). It is then no Sketch to `IsSketch`, so `lym board promote <slug> --recipe tapestry` passes. It keeps `discussed` and its stored record, so D3 seeds a tapestry run from the Sketch, skipping raddle.
   - Fix: either state this as the bound ("a relabelled note is no Sketch"), or key the guard on the stored record or the `discussed` flag rather than the label.
   - Evidence: `internal/boardengine/sketch.go:35-37` (`IsSketch` is label plus kind only).

2. MEDIUM · design · D7 Bound and D9 step 3
   - Summary: the hash check applies only "when the key is off", and the record never says when Raddle-File reads `raddle_auto_approve`.
   - Counterexample: the board refuses an agent's `run_settings` change, but its own refusal notice names `lym config <module> --layer local --set` in the pair as the route for a running pair. `configcli` guards only hub-wide writes by strand name. An agent in interactive mode, or the operator mid-run, can therefore set the key on locally. Raddle-File then files an unapproved or edited proposal with no hash check and no cold-proposal record.
   - Fix: fix the mode at Seed or Raddle-Write's start and record it in the run state, and have Raddle-File read that record. Or extend the bound to name this route.
   - Evidence: `internal/boardcli/runsettings.go:76-78, 87-99`; `internal/configcli/configcli.go:809-813` (only the hub-write guard reads the strand); `internal/configcli/hubwide.go:45-49`.

3. MEDIUM · design · D8 first bullet, last line
   - Summary: "gone from the board counts as landed" is false. Raddle-File then drops a `depends_on` on an entry that was removed unlanded, and the filed part starts without its prerequisite.
   - Counterexample: a weighed task that a part depends on is abandoned with `lym board remove`. The re-check passes it as "landed", and D9 drops the edge.
   - Fix: tell landed from removed, for example by its archive tag or the done status before prune, or name this as what the relaxed guard lets through.
   - Evidence: `internal/boardengine/board.go:224-229` (`RemoveTask` has no done check).

4. LOW · design · D8 second bullet
   - Summary: the check covers only tasks and projects created after the baseline. An existing task outside `board_checked` that gains the Sketch's module label later, or is released from draft into the same packages, passes unseen.
   - Fix: state this as the bound, or compare the labels of entries not in `board_checked`.
   - Evidence: decision-record.md:98; `internal/boardengine/runlock.go:185-217` (labels are scope fields, but the re-check reads only `created`).

5. LOW · design · D9 retry claim
   - Summary: D8's re-check runs inside step 3's board write and requires every new slug to be free. If a crash comes after the write but before anything records it as done, a retry finds the slugs taken and returns Stuck to Raddle-Write instead of skipping. The record names no done-marker for step 3.
   - Fix: name the marker, such as the run's entry already being done, and check it before the re-check.
   - Evidence: decision-record.md:99, 101, 108.

6. LOW · consistency · D9 step 3 and D10
   - Summary: marking the run's own entry done mid-run fires dobby's landing notice when it carries `notify_on_land`, before the beam and the notice steps, and Prune keeps such an entry. "Batten's Land prunes it" therefore fails for that entry.
   - Fix: clear `notify_on_land` in the same write, or state the case.
   - Evidence: `internal/dobbyengine/landnotice.go:105`; `internal/boardengine/store.go:793`.

7. LOW · consistency · D7 second bullet
   - Summary: the waiting notice is posted "on its first pass only". After a D8 return in interactive mode, the new session needs a new approval but posts no notice, so the operator is not told a proposal waits again.
   - Evidence: decision-record.md:84, 102.

8. MEDIUM · design · Open risks, line 3, with D12
   - Summary: whether a Codex rollout splits cache-write is a fact the repository already settles, so the risk defers research.
   - Evidence: the rollout's `token_count` carries `input_tokens`, `cached_input_tokens`, `output_tokens` and `reasoning_output_tokens`, with no cache-write field (`internal/shuttleengine/codexengine/rollout_test.go:23`; `docs/research/codex-review-bench-2026-10-25/report.md:196`). Codex has no `UsageReader` today; it reads only `last_token_usage.total_tokens` (`internal/shuttleengine/codexengine/rollout.go:50-53`).
   - Fix: record the field set and its source in Constraints. Then decide what cache-write means for Codex: zero, not unknown, if the provider has no such charge.

**Tool calls**

- `internal/boardcli/runsettings.go` lines 1-60: whether an agent can set `run_settings` on its own entry; the context lacked the run-settings guard.
- `internal/boardcli/runsettings.go` lines 60-160: the rest of that guard and its local-layer way forward.
- `internal/configcli/configcli.go` around line 800 and its refusal grep: whether a local-layer config write checks the strand; not in context.
- `internal/battencli/land.go` lines 100-135 and a grep for done handling in batten, beam and dobby: what reacts to the run's entry going done mid-run.
- A grep for `Order = append` in `boardengine`: whether filing a member changes a held project's Order, a possible run-lock refusal. Nothing found, so no finding.
