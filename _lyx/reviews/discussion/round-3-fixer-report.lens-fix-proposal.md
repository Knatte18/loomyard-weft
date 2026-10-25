```yaml
lens: fix-proposal
outcome: reported
```

Fork lens fix-proposal: challenge each proposal the fixer carried out (F1–F7) with concrete cases.

Proposals checked: F1 (unruled marks, entry check, ruling on resume), F2 (entry-time gate wording, ways forward at an entered row), F3 (counted-capture predicate), F4 (compare marker as an OS lock, slot-wait bound), F5 (adopt reaches no other machine), F6 (Scope In), F7 (no step reaches `spawnLoop`).

- **MEDIUM — F1, D6 "As the row returns any decision halt under scope `full`, it records … the `unruled` acceptances standing" and "only then does the row mark the recorded acceptances that survive `ruled`".**
  Summary: a resume from any `full` decision halt counts as the parent standing by the acceptances, but only the unruled-acceptance halt names that choice.
  Case: an agent session accepts test A, fixes something else, and its final compare still fails. The handoff ends the row "Stuck at once, naming the report and the ways forward: `lyx drift accept` and resume, or fix and resume" (D6, unchanged). That halt is a decision halt, so the row records A. The parent picks "fix and resume", fixes the code and resumes. The last history entry is that halt, so the row marks A `ruled`. The parent never saw A put to it and was never offered `lyx drift withdraw`.
  The same happens after the dirty-tree and test-gate decision halts.
  Evidence: the changed text says "That halt names the parent's ways forward: `lyx loom resume` to stand by the acceptances … or `lyx drift withdraw`" only for the halt on an `unruled` acceptance. The failing-compare, dirty-tree and test-gate halt reasons name no acceptance.
  A sound change: every `full` decision halt reached while an `unruled` acceptance stands names those acceptances and the stand-by and withdraw ways forward. Or only the unruled-acceptance halt records the set, and the other halts record none.

- **MEDIUM — F3, D5 "The capture `Drift-Verify` counts, the counted capture, is a capture record in state `passed` that, in loom, has `awaited_at` set".**
  Summary: the predicate counts a capture that was still running when a deferral set `awaited_at`, so Webster's code changes can land in the baseline.
  Case: at a `Drift-Await` halt on the branch point, the operator runs `lyx drift capture`. D4 says it "returns once the child holds the capture lock". The operator then runs `lyx drift defer`, which D5 admits at any `Drift-Await` halt and which checks for no running capture, then `lyx loom resume`. `Drift-Await` returns Done on `awaited_at`, so Webster commits code while the capture's command is still running in the same worktree. The capture then ends `passed`, with `awaited_at` set, and `Drift-Verify` compares against a baseline partly taken on changed code. This breaks D4's "no capture ever runs on changed code".
  The HEAD-moved deferral D5 means to count is a capture that ended before the defer.
  A sound change: the predicate requires the capture's end time to precede `awaited_at`. Or `lyx drift defer` refuses while the capture lock is held.

- No finding for F2, F4, F5, F6 or F7.
  For F7, the waiter's non-test callers are `loomcli/drivewiring.go:301` (loom drive), `battencli/drive.go:197` (batten run, which the dobby spawns via `beamcli/dobbydeps.go:245`) and the arbiter relaunch line. None runs as a step child.
  For F1, a goto appends an `OutcomeGoto` history entry (`shedengine/goto.go:135`). So a goto between the halt and the next entry leaves the acceptances unruled and the row halts again, which is safe.

Tool calls:
1. Grepped the `--until-stop` / `RunUntilStop` callers and the shed history appends: needed to test F7's claim about who runs the waiter and F1's history check, which my context did not cover beyond `loopwait.go`.
2. Grepped who invokes `lyx batten run` / drive: needed to confirm the batten waiter is not a step child, which my context did not show.
