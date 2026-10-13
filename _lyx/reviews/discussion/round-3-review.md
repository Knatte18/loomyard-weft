---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:191
    summary: The dropped-notice fallback D16 states (origin/main holding a commit origin/prod lacks) is gated by D17 on an empty deploy key, so on a hub whose deploy key is set, as this hub's is, a dropped landing notice leaves the orch no durable signal that the task landed
    fix: In D17 (decision-record.md:204-206), split the fallback into two checks that each name their durable baseline and neither of which depends on the deploy key. First, at every wake, any task the orch counts as running whose board entry is done, or whose pair is gone, is treated as landed and no longer running, and the newly-ready check below runs for it. Second, at every wake, start any task the board shows ready and not yet started. Keep the origin/main versus origin/prod comparison only as the deploy-recovery check, explicitly scoped to while the landing deploy key is empty and to landings into a require_pr_to_base branch. In D16 (decision-record.md:191), replace the sentence that says the orch learns of the landing from origin/main holding a commit origin/prod lacks with a pointer to D17's board check, and keep origin/prod as the baseline for the deploy only. Add an acceptance row pinning the board-done check sentence in the orch stencil, beside the existing row at decision-record.md:249, and narrow that row to the deploy check.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:132
    summary: D11 says the audit checks every Warn line and that the plan lists every Warn site, and its rationale cites the board ask to check other modules' Warn calls, while the same decision exempts every Warn site outside a loop
    fix: In D11, reword line 132 to "The audit checks every Info-or-above line, Warn included, inside a loop or a polled path against the policy.", reword line 135 to "The plan lists every Info-or-above line, Warn included, that sits in a loop or polled path, each with a verdict (keep, demote or guard), and every changed site has its change in a card.", and replace the rationale at line 137 with one that states the narrowing - the board asks for the other modules' Warn calls to be checked, and the audit reaches the ones in a loop or polled path, where an Info-or-above line can repeat without a state change. Make the D11 heading and the Scope bullet at line 17 read "Info-or-above line, Warn included, in a loop or polled path" the same way. Drop "keeps Warn and" from line 136 so an exempt site is not given a verdict.
---

### [BLOCKING] The dropped-notice fallback disappears when the hub's deploy key is set

**Location:** `_lyx/discussion/decision-record.md:191` (D16) and `_lyx/discussion/decision-record.md:204-206` (D17)

**Issue:** D16 bounds the "not retried" escape hatch by saying the orch learns of a dropped landing "at its next wake from `origin/main` holding a commit `origin/prod` lacks".
D17 runs that comparison only "while the landing `deploy` key is empty".
The two disagree, and the hub this task runs in has `deploy: "./update-plugins.sh"` in `_board/_lyx/config/landing.yaml`.
With the key set, the Land row runs the deploy itself, `tools/deploy/main.go:105` pushes `origin/prod` to the deployed commit, and `origin/main` and `origin/prod` agree after every landing, so the comparison can never reveal a dropped notice there, and D17 does not run it anyway.
The board entry's own incident is exactly this case: the orch reported `intake-bugs-1012` and `cleanup-1012` as still running after an agent-approved landing.
D17's first bullet ("treats the task as landed, no longer running") fires only on a received notice, and no fallback check names a durable baseline for it.
D16 does name one, the board entry turning `done` at teardown, but D17 never tells the orch to read it.
D17's second fallback sentence ("The same wake starts any task the board shows ready") is tied by "the same wake" to the deploy-key-gated wake, so even the ready-task check reads as gated.
The prod baseline also says nothing about a landing into a parent task branch rather than `main`, which D16's "every landing" covers.
The focus question is whether a durable baseline lets the orch tell a missed landing from a handled one without its own context.
It does for the deploy when the key is empty, and not for the landing itself on a hub with deploy configured, which is the hub's shipped configuration.
Round 2's fixer report took `origin/prod` as the baseline for the deploy and is right for that check, but D16 extends it to "learns of the landing", which it cannot carry once the Land row deploys.

### [MEDIUM] D11's audit reach is stated three ways

**Location:** `_lyx/discussion/decision-record.md:17`, `_lyx/discussion/decision-record.md:130-137`

**Issue:** The finish line (line 135) and the exemption (line 136) agree that a Warn site outside any loop is out of the audit.
Line 132 says "The audit checks every Warn line and every Info-or-above line inside a loop", which reads as all Warn lines plus the in-loop Info ones.
Line 135 has the same ambiguity ("lists every Warn site and every Info-or-above line that sits in a loop").
The rationale at line 137 justifies the audit by the board's ask to check "the other modules' Warn calls", the unrestricted ask the decision narrows, without saying it narrows it.
Line 136 also gives the exempt site a verdict ("keeps Warn") in the same sentence that says it needs none.
The acceptance row at line 244 ("No Warn or Info line the plan listed as in a loop or polled path") matches the narrow reading, so the plan writer has two incompatible scopes for one audit.
This is the second focus item's question, and the wording promises checks on sites the finish line exempts.
Round 2's F2 fix applied that review's proposed first sentence verbatim, and that sentence carries the "every Warn line and" ambiguity, so this is what the round-2 fix left behind, not a regression.
