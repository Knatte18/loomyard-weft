# Support log — creel-stage1-fixes

## Interview

The session ran autonomously: every question below was posed and answered by the writer, and each answer is an auto-pick in the question ledger.

### Turn 1 — scope

- Q: Is the task the eight issues plus C1 to C13, nothing else? Recommended: yes; stages 2 and 3 stay notes per the operator's 2026-10-15 decision.
- A (auto-pick): yes. Orch strands stay withheld; `lyx shuttle interrupt`, tender and the orch watcher keep today's interrupt.

### Turn 2 — #657, placeholder read as a draft

- Q: How does a plain `--cmd claude` strand read idle? Recommended: a styled capture, blanking cells in Claude's placeholder style. Alternatives: match the placeholder's wording; have hand-added Claude strands start with suggestions off; change the smoke test to pass settings.
- A (auto-pick): styled capture (D1). Reed is provider-blind, so it cannot add Claude settings; wording matches misread drafts; changing the test drops the "hand-added strand is reachable" claim the board's Done-when keeps.

### Turn 3 — #658, collapsed pane

- Q: Deliver into a 3-row pane, or name the stall? Recommended: name it.
- A (auto-pick): name it (D2), generalised to a draft, since #657 and #659 were both silent stalls behind a box holding text. The TTL keeps running, as `doc.go` states.

### Turn 4 — #659, restored prompt

- Q: Where does the clear live? Recommended: a creel-only shuttle interrupt variant. Alternative: inside `guardedInterrupt` for every caller.
- A (auto-pick): creel-only (D3). Tender's pause and the orch's hub pause interrupt and later resume; changing their box would be an unasked behaviour change.

### Turn 5 — #660, #661, #662

- Q: #660: change `output.Ok` or the read path? Recommended: the read path. A (auto-pick): read path (D4); `output.Ok` has many callers.
- Q: #661: how does the error reach the verb? Recommended: daemon stderr to a per-hub log file the client quotes. A (auto-pick): D5.
- Q: #662: per-inject start? Recommended: charge from the later of previous probe and posting. A (auto-pick): D6.

### Turn 6 — #663, #664, sandbox placement

- Q: #664's release: refuse an unknown name? Recommended: yes, as `hold` does. A (auto-pick): D8.
- Q: Sandbox: a new creel suite file, or the shuttle suite beside S9? Recommended: beside S9. A (auto-pick): D9, as the board entry says.
- Q: Tests for #659 on a real Claude? Recommended: unit tests on the shuttle fake plus C7 in the sandbox; no new llm-tier test. A (auto-pick): as recommended.

### Turn 7 — lens review

All six forks returned a report.

- scenarios, BLOCKING, D3 clear races the receiver's worker during the settle: fixed; the worker neither probes, sends nor records a stall while creel's interrupt runs, and a criterion pins it.
- scenarios, MEDIUM, D3 failure after the key: fixed; logged, interrupt still journaled and reported sent, inject line still queued.
- premises + scenarios, MEDIUM, D2 client-side timeout has no route for the reason: fixed; progress line on the open wait connection.
- premises, MEDIUM, D3 settle via `Runner.InterruptSettle` fails without a run record: fixed; engine resolved as `Runner.Interrupt` does, settle from `InterruptGuard`, criterion for a no-run-record strand.
- pattern-fit, MEDIUM, D3 is an unnamed second clear in `PATTERN-pane-text-injection`: fixed; named in D3 and Constraints, background file added to Scope docs.
- pattern-fit, MEDIUM, D3 clear guarded more weakly than the stray clear: fixed; same three-fact guard reading.
- pattern-fit, MEDIUM, D3 variant not recorded as a `PATTERN-guarded-interrupt` entry point: fixed; named, background file added to Scope docs.
- goal-scope, MEDIUM, tmux tier names only creelcli: fixed; every touched package.
- scenarios, MEDIUM, D5 can quote a stale start error, and an unopenable log is unstated: fixed; start header, quote only lines from the client's own spawn, null-stderr fallback.
- leanness, MEDIUM, Rationales argue against rejected alternatives: fixed; D1 to D3 keep only positive reasons.
- premises, LOW, D5 needs the daemon to raise its stderr threshold: fixed.
- premises + pattern-fit, LOW, D5 path declarer: fixed; `hubgeom`'s creel Geometry, in the state directory.
- premises + scenarios, LOW, D6 time base: fixed; daemon Clock at append, clamped at zero, restart criterion.
- pattern-fit, LOW, D1 widens the empty-box definition: fixed; named in D1 and Constraints.
- scenarios, LOW, D1 mixed box and two-capture timing: fixed; one styled capture per decision, mixed-box criterion.
- scenarios, LOW, D2 stale reason: fixed; reason carries its read time.
- scenarios, LOW, D8 release of a hold on a removed strand: fixed; live hold checked first.
- scenarios, LOW, D4 empty path and partial writes: fixed.
- goal-scope, LOW, D5 warnings beyond #661: fixed; Rationale says why.
- goal-scope + leanness, LOW, D2 draft reason unjustified in the record: fixed; Rationale says why.
- goal-scope, LOW, D8 waiting verbs: fixed; every verb that prints a sender.
- goal-scope, LOW, Scope hides D1's reach to all Claude runs: fixed.
- board-fit, LOW, stage 3 inherits D3's clear: fixed; Open risks.
- board-fit, LOW, D2 wording vs the orch watcher's reason: fixed; plan-writer note.
- board-fit, LOW, `lyx shuttle open` alternative unweighed: fixed; added to Rejected alternatives.
- leanness, LOW, Scope In restates D1 to D8: fixed; one pointer line.
- leanness, LOW, own-board-entry line in Out: fixed; cut.
- leanness, LOW, generic Constraints: fixed; cut comment-line-breaks, test trio and spawn-observability.
- leanness, LOW, test-budget note repeats CLAUDE.md: fixed; cut.
- leanness, LOW, D8 filler Rationale: fixed; cut.

## Rejected alternatives

- #657, match the placeholder's text (`Try "…"`): a draft starting with the same words would read empty and be typed into.
- #657, `lyx reed add` passes Claude settings: reed is provider-blind and stores the provider tag opaquely; it cannot know a command is Claude.
- #657, document `--cmd 'claude --settings …'` and change the smoke test: drops the reachability claim the Done-when keeps.
- #657, hand-added Claude sessions go through `lyx shuttle open`: plain `reed add --cmd claude` is a live path today, which `shuttle-open-permission` records the orch falling back to, since `shuttle open` cannot set the permission mode.
- #659, read the settle through `Runner.InterruptSettle`: it needs a run record, which a hand-added strand lacks.
- #659, let the worker keep probing during the interrupt's settle: its delivery could be read as the restored prompt and cleared.
- #658, make the pane tall enough for the send: moves the operator's layout every delivery; reed collapses stacked strands by design.
- #658, freeze the TTL while too short: an inject into a collapsed strand would sit forever.
- #659, clear the restored prompt in `guardedInterrupt` for every caller: changes tender's and the orch's pause behaviour unasked.
- #659, creel retypes the restored prompt as its own: creel never typed it and cannot know it.
- #661, the client watches its spawned child and reports an early exit at once: a second channel for a speed-up the issue does not ask for; a losing daemon in a start race also exits early, which the client would have to tell apart.
- #661, the client runs the journal load itself before spawning: duplicates the daemon's start and still misses panics.
- D9, a new `SANDBOX-CREEL-SUITE.md` with its own subcommand: the board places the scenarios beside S9.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 scope — resolved, auto-pick: the eight issues and C1 to C13 only.
- Q2 #657 approach — resolved, auto-pick: styled capture with placeholder blanking in claudeengine (D1).
- Q3 #658 deliver or report — resolved, auto-pick: report, for too short and draft (D2).
- Q4 #658 TTL while stalled — resolved, auto-pick: keeps running.
- Q5 #659 layer — resolved, auto-pick: creel-only shuttle interrupt variant (D3).
- Q6 #660 fix site — resolved, auto-pick: the read path (D4).
- Q7 #661 channel — resolved, auto-pick: daemon log file quoted by the client (D5).
- Q8 #662 rule — resolved, auto-pick: from the later of previous probe and posting (D6).
- Q9 #664 release of an unknown name — resolved, auto-pick: refused with the existing names (D8).
- Q10 sandbox placement — resolved, auto-pick: shuttle suite beside S9 (D9).
- Q11 live test for #659 — resolved, auto-pick: unit tests and sandbox C7, no new llm-tier test.
