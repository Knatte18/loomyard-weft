---
dispositions:
  - id: F20
    disposition: fixed-differently
    reason: The bound is twice the startup timeout since the mark, not the startup timeout, because steps 2 to 4 include the relaunched pane's own startup gates, which may take up to the startup timeout, so a bound of one timeout would end a healthy relaunch. Of the proposal's two ways past the bound, the run ends as a strand that died rather than being finished by the reader. Under F21's kept GUID, step 1 ends the process rather than removing the strand, so its failure path is a failure to end the process, which also releases reed's hold. Every record write of the relaunch, not step 4 alone, re-reads the record in one hold of its lock, and both stop forms change to take that lock across their read and write, since the self-check showed a stop overwritten by step 1's clear or by the died end otherwise.
  - id: F21
    disposition: fixed
  - id: F22
    disposition: fixed-differently
    reason: The relaunch spec keeps the run's resolved skills instead of carrying none, and the relaunch launch carries no skill-load turn. Prepare builds the appended system prompt's subagent passage from spec.Skills (internal/shuttleengine/claudeengine/claudeengine.go:269), so dropping the skills would change the relaunched session's system prompt. The prompt offset is set at the relaunch rather than persisted as launch does, since launch persists it only when a PromptLine is sent (internal/shuttleengine/run.go:777-781, 848-860), and the relaunch launch has none.
  - id: F23
    disposition: fixed
  - id: F24
    disposition: fixed
  - id: F25
    disposition: fixed-differently
    reason: The proposed bound "at most once per account before the limit hit's blocked_until rules it out" is wrong. blocked_until is written only when a session on the account hits the limit (D5), so every limited worktree that switches before then can take the account. The text now says switches take it until a session on it hits the limit.
---

# Round 2 fixer-report — discussion

All six findings are fixed in `_lym/discussion/decision-record.md`, and the round is logged in `_lym/discussion/support-log.md`.
`lym loom validate-discussion` passes.
The PATTERN changes bullet's rule text is 242 characters, under the 280 cap.

## Changes

- **F20 (fixed differently).** D7 step 1 records the mark's time.
  A failure to end the process releases reed's hold and clears the mark back to running, and the limit is then handled as today.
  The orphan sweep keeps a run directory whose record reads relaunching.
  A new bound paragraph says an attach or `Wait` reading a relaunching record whose strand is not live ends the run as a strand that died, once twice the startup timeout has passed since the mark.
  A new paragraph has every record write of the relaunch re-read the record in one hold of its lock and leave a terminal record as it is.
  Both stop forms change to take that lock across their read and write.
  When a stop has made the record terminal, the relaunch removes the strand.
  The stop verb removes a strand held for a relaunch whatever its pane state.
  The PATTERN changes bullet is reworded, and three acceptance criteria are added under D7: a killed `Wait`, a sibling sweep and a stop during a relaunch.
  The differences from the proposal are in the frontmatter reason.
- **F21 (fixed).** The first mechanism: the relaunch keeps the strand's GUID.
  Step 1 ends the session's process through reed and keeps the strand, held from reed's reconcile and resume.
  Step 4 has reed replace the strand's stored `Cmd`, `ResumeCmd` and session id, and relaunch its pane under the same GUID.
  A sentence says every caller holding the GUID reaches the relaunched run.
  The Constraints bullet now says the relaunch ends the process through the stop verb's relaunch mode and relaunches under the same GUID through reed.
  A Scope "In" bullet and a plan-writer note name the two new reed ops, since `ReplaceStrand` mints a new GUID.
  An acceptance criterion covers `StopStrand` and attach by the pre-relaunch GUID, and another covers the fresh-launch fallback naming the new session id.
- **F22 (fixed differently).** Step 2 states the relaunch spec: `ResumeSessionID`, the run's resolved skills kept, and one continue line in place of the task prompt.
  The relaunch launch carries no skill-load turn and an empty `PromptLine`.
  Step 4 runs the startup gates (`awaitStartup`), takes the launch's session id and `PromptLine` into the run record, and sends any `PromptLine`.
  It sets the prompt offset at the relaunch, so neither the task pointer nor the limited turn end is read again.
  The no-transcript fallback is a fresh launch with the run's skills, their load turn and the run's prompt.
  D7's first acceptance criterion is extended with the continue line and the startup gates.
- **F23 (fixed).** D6's copy sentence is replaced as proposed.
  A caller that retries a resume after a live-holder refusal, as the orch's resume does, carries the first check's report into the retry.
  The added D6 acceptance criterion covers that retry as well.
- **F24 (fixed).** D7's orch paragraph has both capabilities run the body of the `StartRelaunch` case directly.
  They skip `DecideStart` and the wait for the watcher to go, which would otherwise decide "already running" or stall on the calling watcher.
  The restart's steps are as proposed.
  The fresh start clears the recorded session id, so `launchOrch` launches fresh.
  The orch acceptance criterion is extended.
- **F25 (fixed differently).** D5's bound sentence now covers D7's switch, with the corrected bound in the frontmatter reason.
- **Support log.** A round 2 entry is added under Review rounds, and Q7 now names the same-GUID relaunch and the orch's relaunch branch.

## Self-check

- fix-consistency, MEDIUM: a stop during the window, followed by step 4's failure path, overwrites stopped with died. Fixed: every record write of the relaunch, the died end included, leaves a terminal record as it is.
- fix-consistency, LOW: step 1's sweep clause and the PATTERN bullet disagree on whether the sweep exemption ends at the bound. Fixed: the bullet now bounds only the live reading, and the sweep keeps the directory, as in step 1.
- fix-consistency, LOW: the new reed ops appear in neither Scope nor the seam. Fixed: a Scope "In" bullet names them.
- fix-claims, MEDIUM: the relaunch persists no prompt offset and leaves `RunState.PromptLine` stale. Fixed: step 4 takes the launch's `PromptLine` and sets the prompt offset at the relaunch.
- fix-claims, MEDIUM: "which the stop verb's read and write of the record also take" states as fact a lock the stop verb does not take. Fixed: both stop forms are stated to change to take it.
- fix-claims, MEDIUM: D5's "at most once per account" bound is wrong. Fixed differently from the review's text (see F25).
- fix-proposal, MEDIUM: the step 4 re-read sits only on the success path. Merged with fix-consistency's MEDIUM; fixed there.
- fix-proposal, MEDIUM: the fresh-launch fallback keeps the abandoned session id. Fixed: reed and the run record take the launch's session id, and an acceptance criterion pins it.
- fix-proposal, MEDIUM: the orch's retried resume loses the copy fact. Fixed: the retry carries the first check's report.
- fix-proposal, MEDIUM: dropping the skills changes the appended system prompt. Fixed: the relaunch keeps the skills and carries no load turn.
- fix-proposal, LOW: reed's hold is not released when step 1 fails. Fixed: step 1's failure releases it.
- fix-proposal, LOW: the orch's fresh start would resume the recorded session. Fixed: it clears the recorded session id.
- fix-way-forward, MEDIUM: the fresh-launch fallback keeps the old session id. Merged with fix-proposal's; fixed there.
- fix-way-forward, MEDIUM: the orch restart cannot reach `StartRelaunch` through `DecideStart`. Fixed: the capabilities run the case's body directly.
- fix-way-forward, MEDIUM: step 1's clear can overwrite a concurrent stop. Fixed by the lock-held re-read on every record write.
- fix-way-forward, LOW: a stop after the driving `Wait` dies leaves a held, dead strand. Fixed: the stop verb removes a held strand whatever its pane state.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
