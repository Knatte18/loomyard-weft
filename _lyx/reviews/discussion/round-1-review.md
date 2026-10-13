---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:150
    summary: D13's tmuxkit import of internal/logger closes an import cycle in internal/dotgit's tests, so the Constraints bullet claiming no cycle rests on a false premise
    fix: Rewrite D13 so the discard lives in internal/logger itself, not in tmuxkit — under testing.Testing() the logger's stderr half defaults to io.Discard, the same test-time switch sink.go already uses for the durable half, and a test that asserts on log output still calls logger.SetOutput itself. Drop the tmuxkit.Main sentence from D13 and its trade-off line's reference to tmuxkit, delete the PATTERN-testkit bullet at decision-record.md:208 (tmuxkit gains no import), keep the -v trade-off sentence, and restate the acceptance row at decision-record.md:241 as a package test run whose code logs at Warn leaves no logger stderr output, and the logger package's own tests still pass. Update the Scope bullet Test noise (decision-record.md:18) to name the logger, not tmuxkit.Main.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:135
    summary: D11's finish line requires a per-site verdict in the plan for every production logger.Warn site, several hundred sites, and its acceptance row checks the plan rather than the code, while polled Info lines get no finish line
    fix: Replace the sentence at decision-record.md:135 with a bounded method — the plan lists every Warn site and every Info-or-above line that sits in a loop or polled path, each with a verdict (keep, demote or guard), and every changed site has its change in a card; a Warn site in no loop keeps Warn by D11's own rule at decision-record.md:134 and needs no per-site verdict. Replace the acceptance row at decision-record.md:242 with one observable in code — no Warn or Info line the plan listed as in a loop or polled path logs more than once while its state is unchanged.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:203
    summary: D17 deletes the orch's next-wake deploy fallback while D16 lets a landed notice be dropped unretried, so a dropped notice leaves an empty-deploy hub never deployed and its newly ready tasks never started, with no defined route for the orch to learn of the landing
    fix: In D16 (decision-record.md:189), extend the bound to say what a dropped notice leaves behind and how the orch recovers it — the line is still appended to the hub notice log before the hold-back check, and the orch learns of the landing at its next wake by checking the board for tasks newly done and newly ready. In D17, replace the sentence at decision-record.md:203 with one that keeps a next-wake fallback — at each wake, while the landing deploy key is empty, the orch compares the board's done tasks against the landings it handled and runs update-plugins.sh for any it missed — and add a matching acceptance row that the claims test pins the fallback sentence.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:88
    summary: D5 leaves open how the reason file stays current under the new dedupe — report in innerrun.go returns before reportStuck when the reason text is unchanged, so one gate keyed on the stripped reason would stop the file from holding the newest text — and leaves open how the changing parts are separated from the reason
    fix: Add two sentences to D5 after decision-record.md:88 — the wait report writes the reason file whenever the reason text changes and logs only when the dedupe key changes, and each caller passes that key beside the text (the reason with its counters, countdowns and error tails left out) rather than having the key parsed back out of the formatted text.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:36
    summary: decision-record.md carries forensic and board-fit narrative the plan writer does not act on — D1's quoted tmux error and landing timeline, D16's orch-split sentence, and the first Notes bullet pointing at a trace file the 14-day trace sweep will delete
    fix: The requirement content is already extracted (D1 keeps that neither render nor reconcile changes; D2 and D18 carry the fix). Delete from decision-record.md the parenthetical quote at line 36, the sentence at line 38, the sentence at line 192 and the bullet at line 250, and add each, if not already there, to support-log.md's Interview item 2 (D1 evidence) and item 8's board-fit orch-split entry.
---

### [BLOCKING] D13's tmuxkit import of the logger closes an import cycle

**Location:** `_lyx/discussion/decision-record.md:150`, with the Constraints bullet at `:208`

**Issue:** D13 has `tmuxkit.Main` call `logger.SetOutput`, so `internal/testkit/tmuxkit` gains a non-test import of `internal/logger`.
The Constraints bullet rules out a cycle by looking only at logger's own `TestMain`, which is in `logger_test`.
It misses logger's transitive imports.
`go list -deps ./internal/logger` gives `dotgit`, `lyxcwd`, `lyxdirs` and `proc`, and `internal/dotgit/testmain_test.go` is `package dotgit` (an in-package test) and imports `internal/testkit/tmuxkit`.
With the change, `dotgit`'s test build imports tmuxkit → logger → lyxcwd → dotgit, which Go refuses as an import cycle in test.
The `cmd/lyx` tmux-isolation scan has an empty allowlist (`allowedNoTmuxMain`), so dotgit cannot stop calling `tmuxkit.Main` either.
`internal/logger/sink.go:175` already switches the durable half off under `testing.Testing()`, so the logger itself can apply the same test-time default to its stderr half with no new import.

### [MEDIUM] D11's finish line is a per-site verdict list over every Warn site

**Location:** `_lyx/discussion/decision-record.md:135`, with the acceptance row at `:242`

**Issue:** At this HEAD there are 685 production `logger.Warn` calls across 186 files (`grep -rn 'logger\.Warn(' internal cmd`, excluding `_test.go`).
D11 makes a verdict for each of them a precondition of the audit being done, so the plan has to carry hundreds of verdict entries, most of them "keep" for sites in no loop, which D11's own test at `:134` already settles.
The acceptance row checks the plan document, not any behavior of the built code.
D11 also brings every polled Info-or-above line into the audit, but its finish line and acceptance row cover only Warn sites, so that half has no stopping point.

### [MEDIUM] A dropped landing notice has no recovery once D17 deletes the next-wake fallback

**Location:** `_lyx/discussion/decision-record.md:203`, with D16's bound at `:189`

**Issue:** D16 lets a notice that cannot be queued (orch state unreadable, no orch strand recorded) be dropped with a Warn and no retry.
Its stated bound covers only the batten row and the board entry.
D16's own rationale says the notice is the orch's only prompt that a task landed.
D17 then removes the orch stencil's line telling the orch to deploy "at your next wake after any landing" (`contracts/stencils/orch/orch-template-role.md:130`), which today is the fallback for a landing the orch never heard about.
Together, a dropped notice on a hub with an empty landing `deploy` key means `update-plugins.sh` is never run for that landing, and newly ready tasks wait until something else prompts the orch.
`rowNotifier.send` (`internal/battenshed/rownotice.go:197`) appends the line to the hub notice log before the hold-back check, so a durable record exists, but neither Decision names it or any other route the orch uses to recover.

### [LOW] D5 does not say how the reason file stays current, or where the dedupe key comes from

**Location:** `_lyx/discussion/decision-record.md:88`

**Issue:** `innerRunProducer.report` (`internal/battenshed/innerrun.go:564`) returns before `reportStuck` when the reason equals the last one, so today one gate controls both the log line and the reason-file write.
D5 keys the log dedupe on the reason without its changing parts, and the acceptance row requires the reason file to hold the newest text.
If the existing gate is just rekeyed, the file stops updating, which contradicts that row.
D5 also leaves open whether the stripped key is parsed out of the formatted reason or supplied by the caller.
Parsing it out is fragile across the free-text reasons the producers format.

### [NIT] Forensic and board-fit narrative in the decision record

**Location:** `_lyx/discussion/decision-record.md:36`, `:38`, `:192`, `:250`

**Issue:** D1's quoted tmux error and its landing timeline are evidence for the diagnosis, not something the plan writer acts on.
D16's `orch-split` sentence is board-fit deliberation.
The first Notes bullet points at a hub trace file the exit sweep's 14-day bound will delete, and at another pair's `reed.json`.
None of these carries a requirement that is not already in D1, D2 or D18, so moving them to the support log loses nothing the plan needs.

## Rejected

- D1's premise: confirmed live. The hub watchdog (pid 3854, started 2026-10-08) runs `/home/knatte/go/bin/lyx (deleted)`, and its trace's first `resize re-apply failed` is at 2026-10-12T14:06.
- D4's premise: confirmed. `SetDurableSinkDir` only resets the sink generation, `NotifyExit(0)` returns early, and the sink opens lazily, so a Debug-only exit-0 process writes no trace file.
- D9's coverage: `heldTyping` logs and stores only the first line of a busy error, so the pane tail does not reach the orch, and the two routes D9 names (`readiness.go:93`, `send.go:546`) are the ones carrying draft text into `orch: injection held`, `orch: injection held by the session` and `State.Stuck`.
- D14 and D15 bounds: the collision suffix, partial-entry removal, copy-not-move, unparsable-name and plain-file rules each have a matching acceptance row, and the prune is confined to timestamp directories under the archive root.
- D2's bounds: retire-once, the chain bound, the failed-stat rule and the failed-respawn recovery are each stated. The respawn path resolves the new binary because `os.Executable` drops the ` (deleted)` suffix.
- D4, D6, D7, D8 and D13 diagnostic loss: each either keeps the line in the default trace at Info, or states why the Debug or discarded line is redundant or the trade-off it accepts. None removes a guard, so the attack-surface rule does not apply.
- D15's rationale that a configurable retention needs `HubWide`: holds, since the `logger` module is per-worktree in `configreg`.
