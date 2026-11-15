```yaml
lens: way-forward
outcome: reported
```

Way-forward lens on the plan: for each refusal, stop or recovery step the cards add, I checked whether its named next step can succeed from the state it leaves.

**Findings**

1. **BLOCKING** — Card 3 (Refusal for `ReasonHeld`), card 5 (`Held` read from the adopt pause) and card 14 (`Verdict` sets aside only `Pause` and `Cancel`).
   - Summary: an adopt-paused run can no longer be released by `lym tender resume <slug>`, and the held refusal names a wait that never ends.
   - Evidence: `pattern/PATTERN-dobby-runs.md:21` says "only `lym tender resume` or `lym batten cancel` lifts it, never the adopt check".
   - Card 5 sets `Held` from the adopt pause entry. Card 14's resume `Verdict` keeps `Held`, so `Act` waits with `ReasonHeld` and `Resume` lifts nothing.
   - The held text tells the operator to "wait for the adopt that holds the hub to finish", but that adopt never lifts the entry.
   - Cards 10 and 12 refuse a held run with the same text, so their next step does not progress either.
   - Fix: `lym tender resume <slug>` sets `Hold` aside. While `Held` stands for the adopt pause, the held refusal names `lym tender resume <slug>`.

2. **BLOCKING** — Card 3 (paused Refusal text), card 14 (resume `Verdict`).
   - Summary: a shed `paused` state is refused with a way forward that cannot lift it, and `lym tender resume <slug>` names itself.
   - Evidence: card 3 makes "pause covers and paused name `lym tender resume <slug>` then the re-run". `ReasonPaused` comes from batten's or the child's shed state `paused`, for example after `lym batten goto` or `lym batten pause`.
   - Card 14's `Verdict` sets aside `Pause` and `Cancel`, not `ShedPause`. So `lym tender resume <slug>` on such a run refuses with a text naming `lym tender resume <slug>` again, and its pause entry is never lifted by slug.
   - Only `lym batten run` and `lym loom resume` set `ShedPause` aside.
   - Fix: the `ReasonPaused` text names `lym batten run <slug>`, or `lym loom resume` for the child. Tender resume does not refuse on a shed pause.

3. **BLOCKING** — Card 9 (`cleanupDeps.PairPresent` from local `PairParts(...).Any()`).
   - Summary: `lym batten cleanup --apply` on a machine that does not hold a kept pair deletes that pair's cancelled record.
   - Evidence: D1 says "a kept pair's task branch stays on origin, so its record reads `cancelled` on every machine". D3 says cleanup "refuses one whose pair stands".
   - The batten record lives on the prime's records side, which every machine pulls. A machine without the pair reads `PairParts.Any()` false, judges the record a candidate and commits its removal.
   - The way forward the card names, `lym fabric remove <slug>`, cannot run there either, since the pair is absent.
   - Fix: judge "pair gone" as D1 does, from the task branch on origin (`TaskBranchOnOrigin`), together with the local parts.

4. **MEDIUM** — Card 14 (`pauseCmd` ordering).
   - Summary: a pause of a run another machine owns hits the existing no-pair refusal first, so the owner refusal D3 requires is never reached.
   - Evidence: `internal/tendercli/cli.go:151-153` refuses "`%q` has no pair in this hub; way forward: name one of the pairs" before `tenderengine.Pause` runs.
   - A run owned elsewhere usually has no pair here, so that way forward does not progress. D3's acceptance line wants the refusal to name the owner machine.
   - Card 14 does not move or bypass that check.

5. **MEDIUM** — Card 3 (terminal `Landed` text), card 10 (`refuseByAct` runs before `checkRunStatus`).
   - Summary: `lym batten run` of a slug whose done record still stands now gets "`lym board list` to check it" in place of `doneSlugRefusal`'s steps.
   - Evidence: `internal/battencli/arm.go:848-853` names `lym batten cleanup --apply`, the leftover branch deletion, or a new slug. Card 10 puts the act rule ahead of that refusal.
   - Fix: the `Landed` refusal with a batten record keeps those steps.

6. **MEDIUM** — Card 3 (`Refusal` for `ReasonCancelling`), card 8 (interrupted refusal).
   - Summary: finishing an interrupted cancel as told can silently turn a `--remove` cancel into a keep.
   - Evidence: card 3 says "cancelling names `lym batten cancel <slug>`", with no `--remove`, although the intent records `Remove`.
   - Card 2's `WriteCancelIntent` replaces an unfinished intent with the re-run's flag. So following the way forward keeps the pair the original cancel meant to remove.
   - Fix: when the intent's `Remove` is true, the text names `--remove`. That needs the intent passed to `Refusal`, or carried in `Verdict`.

7. **MEDIUM** — Card 7 step 1 and card 14 (`Verdict` with only `Pause`, `Cancel` and `Hold` aside).
   - Summary: once the later parts fill `PushOwedOffline`, cancel, pause and resume will refuse offline, against D3.
   - Evidence: card 7 says the cancel's verdict refuses "once the later parts fill them, an owed push offline".
   - D3 says that with origin unreachable, `lym batten cancel` "ends the run's processes and strands and commits the intent", and `lym tender resume <slug>` "commits the cleared intent". The prescribed refusal leaves no offline progress.
   - Fix: these verbs set the owed-push wait aside, or the cards state that the later part must.

8. **LOW** — Card 7 (`cancelledRunRefusal` text).
   - Summary: "then release it and run it again" names no command.
   - Evidence: `releaseCommandText` (`internal/battencli/startguard.go:441`) exists, and the other start refusals name it.

9. **LOW** — Card 14 (`Record` and `Clear` errors, and a non-reachability `PullBolt` failure in `Owner`).
   - Summary: none of these errors names a way forward.
   - Evidence: card 14 says a `Record` error "ends the pause wrapped, with the entry live" and a `Clear` error "refuses with nothing lifted", but names no re-run. The re-run does progress, since `Record` runs again on an already-paused scope.
   - A Bolt that cannot be brought level, without being unreachable, returns bare.

10. **LOW** — Card 9 (`Teardown.Run` ordering), with `pairteardown.BattenFollowUp`.
    - Summary: the warning can tell the operator to run `lym batten cancel <slug> --remove` for a cancelled record that `lym fabric remove` already deleted, or is about to.
    - Evidence: `BattenFollowUp` returns the `--remove` follow-up for every state but done and Worktree-Teardown, and `cancelled` is among them. `internal/pairteardown/teardown.go:555-558` computes the warning after the removal.
    - Card 9 does not say whether `removeCancelledRecord` runs before or after the follow-up read, and does not exempt `cancelled` from it.

11. **LOW** — Card 7 (`CommitRecord` through `commitRunRecords`).
    - Summary: with a merge in progress on the records side, `commitRunRecords` commits nothing and returns nil, so the cancel ends the run with its intent uncommitted.
    - Evidence: `internal/battencli/commitstatus.go:104-111`. D3 requires that the cancel "commits and pushes it before it ends anything".

Tool calls: one grep of `pattern/PATTERN-dobby-runs.md` and `internal/pauserecord` for how an adopt pause is lifted. My context held the leaf-packages background, not the dobby-runs one.
