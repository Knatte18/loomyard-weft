# Support log — creel-agents

## Interview

Autonomous run: no operator; every question below was decided by the writer and is listed as an auto-pick in the question ledger.

### Turn 1 — exploration

Read the board entry, the stencil audit's block 7, the creel engine and CLI docs, the parent directive, hold notice, driver notify stencils, seat engine, shuttle send and interrupt verbs, the requirements set and the notice walks.
Found: Go-side notices (hold, batten, Go driver relay) already post through creel to the nearest live orch; the parent directive names the static parent orch; the parent-review delivery stencil the entry names no longer exists, its relay now the Go driver's creel notice; `read` already prints the reply command and shuttle already appends the trust and reply sentence.

### Turn 2 — design batch (self-answered)

- Route for a parent ask: `--parent` resolved at post time (picked) versus the rendered static name.
- Trust of a reply from an orch other than the named parent: widen to any orch strand (picked).
- `lym shuttle send`/`interrupt`: remove (picked) versus wrap creel.
- Seat notices typed by the seat engine: first moved to creel posts, dropped in the lens review (turn 3).
- `RequirementPeerMessaging`: remove (picked).
- Reply discipline: flag `--expect-reply`, two chases at 15 minutes, `unanswered` outcome, report verb (picked).

### Turn 3 — lens review

Six forks returned reports: interactions, existing-paths, bound-breaker, walkthrough, board-scope, leanness.
Duplicates merged; each line names the lenses.

- D1 walk start (walkthrough 1, existing-paths 2, bound-breaker 5): fixed; D1 now pins batten's run-notice rule, `NotifyParent`'s walk from a pair and `NotifyFrom`'s from a beam or the prime.
- D1/D3 same strand only for pair runs (interactions 1): fixed by the same rule; acceptance covers pair and beam.
- D1 refusal never produced by the walk (existing-paths 3, bound-breaker 5): fixed; refusal is "no orch strand recorded at the resolved worktree".
- No stencil sets `--expect-reply` (interactions 2): fixed; D2 and D5 mark every question.
- Chase timer has no wait to fold into (interactions 3, bound-breaker 2): fixed; D10 adds a timer per receiver, at most once a second; constraint reworded.
- Pause or hold does not stop the chase clock (interactions 4, bound-breaker 3, existing-paths 5): fixed; the clock counts only while the holder is live, unpaused and unheld.
- Holder gone after reading (interactions 5, existing-paths 4, bound-breaker 3): fixed; `unanswered` at once with reason `holder-gone`.
- Chase and program mail versus "trust no other session" (interactions 6): fixed; D2 trusts lym programs.
- `unanswered` outcome for `--wait-reply` callers (interactions 7): fixed; CLI exits 1 with it, Go client returns a distinct outcome.
- Seat testdata fixtures escape the scan (interactions 8, bound-breaker 8): fixed; D13 scans testdata prompts.
- D6 seat notices program claim refused from a driver pane (walkthrough 2, bound-breaker 1) and its landing readers (existing-paths 7): fixed by dropping the decision; the typed up line works on any engine with a pane, so moving it gave no engine-neutrality and added an identity risk. The PATTERN-pane-text-injection change it needed is dropped too.
- D2 off-Linux Codex can pose as an orch (bound-breaker 4): fixed; the bound now states the gap and that it already applies to the parent-name trust.
- Appended system-prompt sentence says to obey all creel mail (existing-paths 1): fixed; new D6 rewords it.
- Never-read marked message is never chased (walkthrough 3): fixed; bound stated.
- `unanswered` notice to a program that never read (walkthrough 4, bound-breaker 7): fixed; journal record alone.
- Mark not in the log (walkthrough 5): fixed; D9 and D11 put it on the post's log entry.
- Reply to a chase bypasses the holder rule (bound-breaker 6): fixed; only the current holder's reply is re-addressed.
- Forward clause acts on nothing (existing-paths 6): fixed; "the expectation follows the holder".
- Restart burst after downtime (existing-paths 8): fixed; overdue steps one interval apart.
- Operator's 2026-10-27 wait (board-scope 1): kept under Open risks as a parent question.
- SendMessage-works-well view unanswered (board-scope 2): fixed; Scope states the trade-off.
- "Move behind creel" read as removal (board-scope 3): fixed; Scope says so.
- Parent-review gate item not stated (board-scope 4): fixed; Scope lists it as already met.
- Auto-mode assumptions repeat decisions (leanness): fixed; trimmed to the two no decision carries.
- Generic PATTERN constraints (leanness): fixed; trimmed to design facts.
- Cycle stated twice (leanness): fixed; Note dropped.
- Codex messaging open risk (leanness): fixed; moved here: whether Codex sessions can message each other is untested and this design does not need it.
- Docs list in Scope (leanness): fixed; kept only the creel doc section the criterion pins.
- D5 near site enumeration (leanness): fixed; trimmed to the verb per role.

## Rejected alternatives

- Agents post to the rendered `{{.parent_name}}`: a dormant or never-started beam orch would hold or refuse the question while the hold notice reaches another orch.
- Creel forwards mail for a dormant orch up the beam chain: creel knows no beam geometry, and the walk already exists client-side.
- Trust only the orch the `--parent` post resolved: the agent would have to remember the envelope's receiver across turns; the orch set is already the operator's delegates.
- Keep `lym shuttle send`/`interrupt` as creel wrappers: two names for one route, guid addressing that creel does not have.
- Keep `RequirementPeerMessaging` for a future need: nothing would require it.
- Post the seat engine's up and ending lines through creel: the step process may run in a driver's pane, where creel refuses a program claim, and the typed line already reaches any engine's pane.
- Deny the `SendMessage` tool by hook: the webster fork addressing needs it; a scan test is enough.
- Chase interval as a creel config key: creel deliberately has no config file.
- Each engine's own messaging with relay agents between engines (operator idea 2026-10-23): creel already types into any pane, so relay agents add two model turns per message, and hook delivery removed the read turn for Claude.

## Review rounds

_No rounds yet._

## Question ledger

- Auto-pick: `--parent` resolves with batten's run-notice rule at post time; settles the board's per-recipe risk.
- Auto-pick: trust widens to any orch strand and lym program of the hub.
- Auto-pick: the appended creel trust sentence defers to role rules for non-parent mail.
- Auto-pick: remove `lym shuttle send` and `interrupt`.
- Auto-pick: seat notices stay typed lines.
- Auto-pick: remove `RequirementPeerMessaging`.
- Auto-pick: flag name `--expect-reply`; `--wait-reply` implies it; every stencil question carries it.
- Auto-pick: chase interval 15 minutes, two chases, then `unanswered`; constant, not config; clock frozen like an inject TTL.
- Auto-pick: a reply to a chase by the holder answers the chased message.
- Auto-pick: `lym report replies` as its own verb.
- Resolved by research: no creel `defer` verb exists, so whether a deferral counts as an answer is moot.
- Open for parent: whether the 2026-10-27 real mid-turn run wait is met (Open risks).
