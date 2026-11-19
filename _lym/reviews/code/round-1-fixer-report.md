---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Proposed: cap the Code-Review ask's wait as CapParentReviewWait caps it, by review_timeout_min. Not made: the Bouncer's judge runs under Spec.Timeout zero, which shuttle replaces with run_timeout_min (shuttleengine/attach.go:405), so a wait capped by the chair's 240-minute review_timeout_min outlives a judge limited to 30 minutes. The seat's wait is capped by run_timeout_min through the new CapParentJudgeWait. The rest follows the proposal: the ask moves to the Bouncer's judge call, after the round's fixer report exists and before the judge spawns, names that round's own review and fixer report, and the judge is told to await the answer."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: ruling
    reason: "The parent ruled that the four parent-ask keys stay CategoryNone and settable in run_settings. Carried out by amending PATTERN-config-layers with the written exception for a key that only adds an ask to the parent and the timeout bounding that added wait. The keys' categories needed no change. Not flagged to the parent."
  - id: F4
    disposition: ruling
    reason: "The parent ruled that judge_ask_parent and judge_parent_timeout_min stand for the entry's drift_judge_ask_parent, with no rename. The code already names the keys so, and nothing needed to change. Not flagged to the parent: the ruling keeps the names, it does not ask for a change the target already holds."
  - id: F5
    disposition: fixed-differently
    reason: "Proposed: cap the deadline by the judge session's OpenedAt plus judgeTimeout less a margin. Not made: shuttle's attach to a live chair starts a fresh deadline of now plus Timeout (shuttleengine/attach.go:358-361), so a resumed judge has its full timeout again and the first OpenedAt would skip the parent wrongly after a resume. The cap is taken from the start of the judge run of the pass, DriftVerify.judgeStartedAt. The rest follows the proposal: the Warn names judge_parent_timeout_min, a limit with no room records the ask not-sent and passes, and TestParentAgreementGate pins the capped deadline and the no-room row."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "Proposed: a round-1 row asking with no prior ask and expecting the bare subject. Not made: F1 moves the ask to the Bouncer's judge call, and every round, the first included, now names its own review and fixer report. TestParentAsker_BeforeJudgeAsk_NamesTheRoundsOwnReviewAndFixerReport is table-driven over rounds 1 and 2 and pins that line, and the producer test pins that a seat asking before the judge asks nothing at round start."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Fixer report, round 1

## What changed

Commits on this branch, oldest first.
Findings that touch the same files share a commit, because a per-finding split would need partial staging.

- `pattern: carry out the parent's F3 ruling` — `pattern/PATTERN-config-layers.md`.
- `loomcli: fix F12` — the not-latest refusal names no decision record; the `parent-review` help text no longer says "beside the reviewer".
- `loomcli: fix F11` — `TestDriftDeps_BuildTheJudgesParentSeat` pins the judge's seat: files under the anchor, the run's slug, both seams set.
- `burlerengine: fix F13, F14` — `ParentAsk` and its `Seed` doc cover the drift judge; asks carry `Tag` and `Digest`; `LatestTagged` and `LatestOfSeed` look an ask up by seat tag and by seed.
- `driftengine: fix F2, F5, F7, F8, F9` — the parent agreement entry:
  - It records the digest of the rulings each ask presented, and a judgement that differs gets a new ask instead of a pass.
  - An open ask on other rulings is set timeout in its place.
  - A findings answer fails while the judgement still holds the answered rulings, replacing the mtime check.
  - The deadline is capped by the judge run's timeout, with a not-sent record when no room is left.
  - New rows in `TestParentAgreementGate` for re-post, a Resolve error, the answer race, the capped and no-room deadlines and changed rulings.
  - `tableWithParent`'s order is pinned.
  - The keep marker is on `TestJudgeAskLine`.
- `shedadapters: fix F1, F6, F8, F10, F14`:
  - The Code-Review parent checkpoint moves to the Bouncer's judge call.
  - `parentAsker` splits the seat's ask work from the review producer.
  - `ParentReviewSeat.AskBeforeJudge` replaces `AskEachRound`.
  - The judge's prompt carries a block telling it to await the answer.
  - The Bouncer settles the ask once the judge has ruled.
  - `shedrecipe` passes the segment's seat to the Bouncer.
  - `TestParentSeats_BuildEachSeatFromItsOwnPerch` pins each production seat.
  - A discussion seat reads only its own asks, so an ask of another seat as the latest no longer repeats its ask.
  - The keep markers are on `TestParentAskLine` and `TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments`.
  - Docs: `orch-role.md`, `docs/overview.md`, the `code_review_*` template comments, `doc.go` of `shedadapters`, `loomcli` and `driftengine`.
- `shedadapters: fix self-check findings` — see the self-check below.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.
All four forks returned a report.

- fix-consistency, LOW: `parent-review` help text says "beside the reviewer" — fixed.
- fix-consistency, LOW: `orch-role.md` says the drift ask goes once per judge spawn — fixed.
- fix-consistency, LOW: drift `template.yaml` comment omits the new ask for rewritten rulings — fixed.
- fix-affected-set, LOW: asks recorded before the tag existed carry none, so the discussion seat asks once more on an in-flight run — fixed: the discussion seat reads untagged asks as its own, with a row.
- fix-proposal, MEDIUM: the Bouncer judge can reach its run timeout while it waits for the parent — fixed: `CapParentJudgeWait` caps the code seat's wait by `run_timeout_min`, with tests in `loomengine` and `loomcli`.
- fix-proposal, MEDIUM: nothing enforces that parent findings rule out CONVERGED — fixed: the Bouncer refuses a CONVERGED verdict while the parent's report holds findings and judges the round again, with a test row.
- fix-code-correctness, LOW: the cap by the session's first open time can skip the parent after a resume — fixed, as F5 records.
- fix-code-correctness, LOW: a digest of the raw file bytes counts a whitespace-only rewrite as changed rulings — fixed: the digest reads the parsed rulings, with a row.

## Disputed

None.

## Deferred

None.

## Rulings

- F3, ruling from the parent (decision notice `Opt-in parent-ask keys stay outside Guard`):
  - It asks to keep `code_review_ask_parent`, `code_review_parent_timeout_min`, `judge_ask_parent` and `judge_parent_timeout_min` settable in `run_settings`, and to write the exception into `PATTERN-config-layers`.
  - Carried out: the Guard bullet of `pattern/PATTERN-config-layers.md` now excludes a key that only adds an ask to the run's parent and the timeout bounding that added wait.
  - No config code changed. Not flagged to the parent.
- F4, ruling from the parent (decision notice `Drift key names stand`):
  - It asks to keep `judge_ask_parent` and `judge_parent_timeout_min` and not rename them.
  - Carried out by leaving the names as they are. Not flagged to the parent.
