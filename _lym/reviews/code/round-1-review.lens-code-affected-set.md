```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change I list the set I found, then the findings.

**Per change: set found, covered or not**
- **Halt notices carry autonomy** (`NotifyParentHalt`, `NotifyRunFromHalt`). The diff stamps these call sites:
  - `internal/loomcli/drivewiring.go:258` (hand-back).
  - `internal/loomcli/prepush.go:79`.
  - `internal/battencli/notices.go:27-31`.

  Other posters I found are unstamped:
  - `internal/beamcli/dobbydeps.go:186`, `NotifyRunFrom` for dobby's per-run notices.
  - `internal/loomcli/cli.go:211`, `landingdeps.go:279`, `wiring.go:267` and `wiring.go:995`, `raddlewiring.go:246`, `raddleverbs.go:248`, and `darnwiring.go:272`.

  Partly covered. I could not confirm which of the unstamped ones are halt notices (see F1).
- **`autonomy` key in `beam.yaml`.** Template, `ConfigSpec`, `LoadConfig`, `CheckLocalValues`, `beamsetting`, `boardcli` help, `configreg` test, the `config-layers` pattern file and the `PATTERN.md` index line are all covered. The prose docs that name the per-task settings are not covered (F3).
- **Goto `Admitted` hook.** Covered on every loom path: `arm.go:479`, `arbiterconsume.go:30`, `drivewiring.go:290` and `shedverbs`. Not covered on the batten paths (F2).
- **`RenderPRReviewPrompt` signature and `parentReviewRequestPrefix` rename.** The only caller outside `landingshed`, the `shuttleparity` test, is updated. No stale reference to the old prefix remains. Covered.
- **PR-Review stencil split into two blocks.** The registry in `contracts/stencils/stencils.go` and the `review_mode` marker are done. The only other references to the stencil name are research docs. Covered. No rule the answer key kept is dropped: the operator-mode text moved whole into the operator block, and the "operator" wording became "reviewer".
- **`ParentAsk.Cleared`.** Readers are `agreementGate`, `ReadPRReviewStanding` and `LatestTagged`. `heldJudgeAsk` (`judgeparent.go:82-88`) still counts a cleared ask as held. I judge that benign: it is only a held check, and the gate resets `held`. Covered.
- **`pr_review_parent_timeout_min`.** Template, `ConfigSpec`, validation and the `overview.md` mention are done. Covered.
- **PATTERN.**
  - The `config-layers` and `daemon-wakeups` pattern files are updated.
  - Declared PATTERN changes: the constraints section declares none, so nothing is owed.
  - Departures: `git log` showed no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers.

**Findings**

- **F1 — MEDIUM, scope.** Location: `internal/beamcli/dobbydeps.go:186`. Dobby's per-run notices post through unstamped `orchcli.NotifyRunFrom`.
  - D2 and the D2 acceptance criteria ask that every halt notice of a run carry `autonomy:`.
  - The diff added `NotifyRunFromHalt` for exactly this case and used it in `battencli/notices.go`, but this site was not converted.
  - Evidence: `dobbydeps.go:186` against `notify.go:44`.
  - Fix: the fixer must confirm whether `runNotices.Post` carries halt notices. If so, use `NotifyRunFromHalt(worktree, slug, dobbyProgram, runNoticeTags(slug), line)`. Also check `loomcli/cli.go:211` (`c.notifyParent`) and `loomcli/darnwiring.go:272` (verify-budget notify) for halt use.
  - I did not see these callers' use sites, only the lines above.

- **F2 — MEDIUM, scope.** Location: `internal/battencli/drive.go:602` (`battenConsumeAttempt`), `drive.go:320` and the batten verb spec at `battencli/arm.go:464`. Batten's goto and relaunch paths pass no `Admitted` hook.
  - D4 requires a goto onto PR-Review to clear the recorded autonomy and session.
  - The loom paths pass the hook. Batten's arbiter consume (`shedarbiter.ConsumeAttempt` with a `RelaunchRequest`) and its tackler consume do not.
  - A batten-driven relaunch targeting PR-Review would keep a stale recorded autonomy and resume the old session.
  - Evidence: `drive.go:595-602` against `loomcli/arbiterconsume.go:30`.
  - Fix: pass `Admitted` on the batten `RelaunchRequest`s and the batten goto spec, or record the omission as intended if batten can never target PR-Review.

- **F3 — LOW, consistency.** Location: `contracts/orchrole/role-beams.md` and `plugins/ly/skills/board/SKILL.md`. Both name the per-task settings `pull_request` and `land_notice` and do not name `autonomy`.
  - The `config-layers` pattern file and the `boardcli` help were updated to list it, so these two are stale.
  - Fix: add `beam.autonomy` where they list the per-task beam settings, as the `boardcli` help does.

- **F4 — LOW, consistency.** Location: `docs/overview.md:948` and the Beam-Await section.
  - The diff's `overview.md` edits cover only PR-Review, `parent-review ask` and `pr_review_parent_timeout_min`.
  - Line 948 still says the turn-end hook posts each turn's final reply to the channel, which is wrong while autonomy is on (D8).
  - The `autonomy` key, the halt stamp and the `[beam drained]` line (D6) are not described there either.
  - Fix: add one sentence each at line 948 and in the Beam-Await description.

**Tool calls (4, all Bash)**
1. Greps for `NotifyParent` and `NotifyRunFrom` callers, `RenderPRReviewPrompt`, stencil name references, goto/relaunch call sites and `heldJudgeAsk`; the manifest does not show callers outside the diff.
2. Greps for the `parent-review` verb lists, the timeout key mentions, the old `gateRetryHint` text and `pr-continue` doc mentions, to find docs and fixtures the diff does not touch.
3. Read the context of each unstamped notify site and of batten's goto and consume code, to judge halt versus non-halt and the `Admitted` gap.
4. Greps of `docs/overview.md`, `PATTERN.md` and other docs for autonomy and per-task setting mentions, to find stale prose.
