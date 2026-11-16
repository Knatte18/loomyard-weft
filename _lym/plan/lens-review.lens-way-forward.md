```yaml
lens: way-forward
outcome: reported
```

I found 11 way-forward defects: 5 where the next step cannot progress, 6 where a refusal or stop names no next step or none is specified.

1. **MEDIUM — card 6 (D1, Webster corrupt state):** a corrupt Webster `state.json` refuses naming "repairing the file, or `lym webster reset --to start`". The second step cannot succeed: `lym webster reset` loads the state first and fails on the same corrupt file.
   - Evidence: `internal/webstercli/reset.go:95-97` calls `websterengine.LoadState` and does `return fail(err.Error())`; `internal/websterengine/state.go:195-198` returns a malformed file as an error.
   - Fix: name repairing the file, or removing it so the run counts as never begun.

2. **MEDIUM — card 6 (both implementers begun):** the both-begun refusal names "set `tapestry.implementer` back" as one way forward. That step cannot clear it, because the seam picks by begun batches before the setting, so both begun refuses again under either setting. Card 17 reuses this text on both rows.
   - Evidence: card 6's own `Active` order (one begun wins, otherwise both begun refuses); card 17's "A seam refusal (both begun…) is returned as the refusal with its own ways forward".
   - Fix: for both begun, name only the resets.

3. **MEDIUM — card 6 (Weaver corrupt-state repair advice):** the advice must name the commit the restorable version came from, but the planned history read cannot supply one. `CommittedAnchoredFileHistory` returns `[][]byte` with no commit. Cards 6 and 14 carry the same `StateHistory`.
   - Evidence: `internal/fabricengine/committedfile.go:110` (`CommittedAnchoredFileHistory`).
   - Fix: `AnchoredFileFirstParentHistory` returns `AnchoredFileVersion{SHA, Position, Data}` (`committedfile.go:47-55`), and the first-parent line matches "the per-batch weft commits".

4. **MEDIUM — card 20 (renumber over a seam refusal):** a seam refusal (both begun, or a corrupt state) "joins the raw-I/O refusal". That maps its way forward to "transient, re-run the refused verb; nothing is mutated", and re-running hits the same refusal.
   - Evidence: `internal/loomcli/renumber.go:161` (the `errRenumberRawIO` row of `renumberWaysForward`); card 20's first bullet.
   - Fix: give the seam refusal its own row, carrying the seam's ways forward.

5. **MEDIUM — cards 12 and 13 (plan-conflict stop, then re-entry):** card 12 has `stopReason` record a `StopRecord` for every stop, and card 13 runs `reenter` whenever one is set. Nothing excludes the plan-conflict Stuck. Two failures follow on the re-step after Plan-Burler-Review:
   - (a) If the conflict was recorded as a batch stop, `reenter`'s no-spawn path runs the post-batch checks on a batch no attempt ran. An Edit-only batch passes the done checks on a clean tree and is recorded done unbuilt.
   - (b) Plan-Review fixing the conflict by inserting or removing a pending card makes `reenter` refuse, because the recorded partition holds the old pending batches. The refusal names "put the card set back or `lym weaver reset`", which undoes the reroute's own fix.
   - Evidence: card 12 bullets 2 and 5; card 13's `reenter`; D3 records pending batches only until the next begin replaces them.
   - Fix: state that a conflict Stuck records no `StopRecord` and that the next call repartitions from the current plan.

6. **MEDIUM — card 14 (reset, commit failure after the archive):** if `Commit` fails after the archive, a re-run finds `state.json` already renamed, and card 14 defines no reset over an absent state. The restored plan and the archived record stay uncommitted with no named way forward.
   - Evidence: card 14's step order (move, restore, archive, clear, commit) and its refusal list (lock, corrupt, probes), which has no absent-state branch.

7. **MEDIUM — card 14 (no dirt check before the branch move):** `Reset` moves the branch through `ResetCode` with no check for tracked changes the run did not write. Webster refuses those with "commit them … or restore each with `git checkout -- <path>`", but Weaver's reset would discard the operator's uncommitted edits silently.
   - Evidence: `internal/websterengine/reset.go:238-251` (Webster's foreign-dirt refusal); `ResetPairCode` resets hard (`internal/fabricengine/destroy.go:2324`).

8. **LOW — card 14 (reset with no state or no begun batch):** there is no refusal when the state is absent or holds no begun batch, so there is no start commit to move to. Webster refuses "the run has no state.json … run `lym webster run` first".
   - Evidence: `internal/websterengine/reset.go:150-152`; card 14's refusal list.

9. **LOW — card 14 (`--stop-strand` stop failure):** under `--stop-strand`, a found run whose handle `Stop` fails has no stated outcome or way forward. Elsewhere the plan names `lym reed remove <guid>` for a failed stop.
   - Evidence: card 14 bullet 3, "stops every found run through the handle's `Stop` and goes on".

10. **LOW — cards 12, 16 and 10 (stop texts with no way forward):**
    - `ErrRunBusy` is mapped to Stuck "with the error's text", but card 12 gives that text no way forward. Webster's names "wait for the run to finish, then retry" (`internal/websterengine/archive.go:109`).
    - Card 10's per-batch commit failure "is returned" and card 12's final commit failure is unspecified, both with no way forward. Webster's wrapper words it "transient, re-step the row" (`internal/loomshed/webster.go:98`).

11. **LOW — card 15 (refusal outside a hub):** `Command` "refuses outside a hub" with no way forward named (PATTERN-refusal-way-forward).

Outside this lens: card 21's `notifyLiveRuns` holds strand names (`decision.go:473-478`), but `RecordDelivery` takes a strand guid. The card names no mapping from name to guid.

Tool calls:
- `internal/fabricengine/committedfile.go:44-56`: to check whether `AnchoredFileVersion` carries a commit SHA. My context had only the function signatures.
- `internal/webstercli/reset.go`, the `LoadState` lines: to confirm Webster's reset fails on a corrupt state. My context had only the subagent's summary.
