```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens. For each refusal, stop or recovery step a card adds, I checked whether the next step it names can succeed from the state it leaves.

## Per refusal: the next step, and whether it progresses

- **Card 4**, `LoadConfig` refusing a negative `beam.yaml` value → `lyx config beam --set`. Progresses: configcli has `--set`, and `beam` is registered on the same card.
- **Card 4/32**, dobby state record that does not decode → none named. Does not progress (F1).
- **Card 7**, draft start refusal → `lyx board set-status '{"slug":"&lt;slug&gt;","status":null}'`, then `lyx batten run &lt;slug&gt;`. Progresses: card 5's rule reads draft→nil as a release, so the task stays released.
- **Card 7**, existing refusals whose way forward is a cancel followed by a fresh `lyx batten run` (left-pair refusal, no-seed refusal) → the fresh run now meets the draft refusal first. Progresses, but only through a second refusal (F4).
- **Card 15**, driver-dead after the respawn cap → `lyx loom start` in the task worktree. Progresses: the new driver writes the status, which resets the count, and the next pass adopts the live driver.
- **Card 15**, `driver strand could not be revived` → none named. Partly progresses: the halt notice sent before it names the resume (F3).
- **Card 17**, records-stop line → no `way forward:` clause of its own; it relies on the `&lt;stop&gt;` text. Progresses only if that text carries one (not flagged).
- **Card 18**, `RowStopNotice` → fix the cause, then `lyx batten run &lt;slug&gt;`. Progresses: the hand-back leaves the run blocked, and a hand pass resumes it. The pause-honoured Land stop keeps its own reason naming `lyx batten goto &lt;slug&gt; --to Land`. Progresses.
- **Card 24**, batten arbiter refusals → `BattenHandBackWayForward` (`lyx batten status &lt;slug&gt;`, hand back to the parent). Progresses.
- **Card 25**, `battenConsumeAttempt` given a bad attempt number → hand back to the run's parent. Progresses.
- **Card 26**, owned-elsewhere refusals → `lyx adopt &lt;slug&gt;`. Progresses. Unreachable-origin refusal → re-run once origin answers. Progresses. Covered refusal → `lyx resume`. Progresses.
- **Card 28**, `StartFirstPass` refusals (done slug, covered slug) → their existing ways forward. Progresses. Dobby-spawn failure → `lyx beam start`. Progresses once card 37 has landed.
- **Card 29**, marker write failure → the cause and a re-run. A plain re-run fails again while the obstacle stands. It is exempt as a propagated raw I/O error (not flagged).
- **Card 30**, `continueRefusal` → `lyx batten run &lt;slug&gt;`. Progresses: dobby reads exit 79 as a full pass owed, and a hand pass is a full pass.
- **Card 33**, failing-pass notice → fix the cause, then `lyx batten run &lt;slug&gt;`. Progresses, and dobby keeps retrying.
- **Card 34**, owner notice → `lyx adopt &lt;slug&gt;`. Progresses: adopt lifts the owner pause, which gets the run its full pass. A beam slug goes through card 39's fixed-point call.
- **Card 35**, refused-start notice → the batten refusal's own text. Progresses where that text names a step. After an adopt, the entry change retries the start at once.
- **Card 27**, `EnsureLive` on a beam owned elsewhere → skipped, nothing named. Does not progress for the beam (F2). The prime case names `lyx adopt --prime` and progresses.
- **Card 36**, tidy timeout posting that no report came → none named (F5).
- **Card 37**, create/promote dobby-start failure → `lyx beam start`. Progresses. Held lock → `already_running`. Progresses: nothing is needed.
- **Card 38**, orch-start dobby failure → a Warn naming `lyx beam start`. Progresses.
- **Card 39**, adopt refusals → the named fixed-point call. Progresses for local commits (`--keep-local`). For a live pair the next step is not a verb (F6). The `dobby_warning` names `lyx beam start` and progresses.
- **Card 40**, `ErrRunRunning` on reed down of a dobby-driven member → `lyx tender pause &lt;slug&gt;`, then reed down again. Progresses: the pause makes the pair read settled.

## Findings

- **F1** — MEDIUM, class design. Location: card 4 (`ReadState`/`UpdateState`), with card 32 (tick loop) and card 31 (`dobbyReading`).
  Summary: a dobby state record that does not decode wedges the beam with no way forward.
  - `UpdateState` reads before it writes, so every tick fails. Card 32 only logs that once per episode and goes on.
  - Card 31 returns `dobbyReading`'s read error and does not say who absorbs it. If the error reaches `wire`, every hand pass of every run under that parent is refused as well.
  - Card 13, by contrast, reads its own undecodable memory record as empty with one Warn.
  - Hand-editing a state file is no way forward per refusal-spec.
  - Evidence: `_lyx/plan/31-batten-pass-wiring.md:23`, `:27` ("an undecodable record returns its read error"); `internal/state/state.go:130` (ErrDecode on unmarshal); `_lyx/plan/32-dobby-loop.md` (tick failure logged per episode, loop goes on); `_lyx/plan/13-innerrun-short-check.md` (`loadMemory` reads undecodable as empty).
- **F2** — LOW, class consistency. Location: card 27 (`EnsureLive`).
  Summary: a beam owned elsewhere is skipped silently, while the prime case reports its owner and `lyx adopt --prime`. The operator gets no step that starts that beam's dobby (`lyx adopt &lt;beam&gt;`).
  - Evidence: `_lyx/plan/27-dobby-start.md:26`.
- **F3** — LOW, class consistency. Location: card 15 (`conditionText`).
  Summary: the new driver-dead text for a failed revive, `driver strand could not be revived`, names no next step. The cap text on the same card names `lyx loom start`.
  - Evidence: `_lyx/plan/15-innerrun-respawn-cap.md:20`.
- **F4** — LOW, class consistency. Location: card 7.
  Summary: existing refusals whose way forward is a cancel followed by a fresh `lyx batten run` now meet the draft refusal, because card 7 leaves a cancelled task `draft`. The step still progresses through the draft refusal's own way forward, but the old texts no longer name the release.
  - Affected texts: the left-pair refusal ("run `lyx batten cancel &lt;slug&gt; --remove` ..., then `lyx batten run &lt;slug&gt;`") and `preDriveCheck`'s "run `lyx batten run &lt;slug&gt;` for a fresh start".
  - Evidence: `internal/battencli/startguard.go:160`, `internal/battencli/arm.go:711`.
- **F5** — LOW, class scope. Location: card 36 (`tidyStep` timeout).
  Summary: the "no report came" line names no next step. The card does not say whether the landed slugs since the last tidy carry over to the next tidy, so those landings may never be tidied.
  - Evidence: `_lyx/plan/36-dobby-board-tidy.md:31`.
- **F6** — LOW, class design. Location: card 39 (`adoptRefusal`).
  Summary: for a reached entry with a live pair, the named call "carries ... the end of the pair's driver". That is no flag and no verb, so the named call does not pass step 1 as typed, and the refusal names no command that ends the driver.
  - Evidence: `_lyx/plan/39-adopt-refusals.md:21`; `internal/adoptcli/adopt.go` `takeOverSlug` live-pair stop.

Tool calls:
1. A grep for line evidence: the startguard and arm refusal texts, `ErrDecode` in `internal/state`, configcli's `--set` flag, and the card lines quoted above. My context lacked exact line numbers and the configcli and state internals.
