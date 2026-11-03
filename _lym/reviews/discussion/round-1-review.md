---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (report and end marker path)
    summary: The assistant's report path under the anchor's .lym lies outside the Codex workspace-write sandbox, so the assistant can never write its report
    fix: In D2, replace the report location with one inside the warp worktree's real (non-symlinked) tree that the workspace-write sandbox covers, or a writable root the spike tested (/tmp or $TMPDIR, under a lym-owned per-round directory), and state the evidence (spike report.md lines 10 and 283; the worktree's .lym and _lym are links into the weft checkout, where .lym-anchor sits). Drop the claim "so the workspace sandbox covers them" unless the new path makes it true; keep the Go copy beside the review file. Update the D2 acceptance bullet to name the new location, and add the fact to Constraints (the anchor's .lym and _lym resolve outside the warp worktree, so Codex cannot write there).
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 and D2 (stale marker and outputs at stable per-round paths)
    summary: A second attempt of the same round reuses the first attempt's marker and report, which are cleared only before the assistant starts, after the chair is already live, so the chair or a resume probe can read the old attempt's ending
    fix: In D1, state that the table builder removes a stale marker and stale outputs before the chair starts, and that a probe ignores outputs older than the current attempt; or key the marker and report paths on the review row's attempt as well as the round. Update the matching D1 acceptance bullet (a fresh run removes a stale marker and stale outputs before the chair starts; a retry's chair never reads the prior attempt's marker).
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D12, D6 and Open risks (heddle entries on claude-file alone)
    summary: Every heddle entry, shipped and repository, targets claude-file alone and the repository has no CLAUDE.md, so the Codex assistant gets none of the instructions the chair gets, against D12's purpose and the operator's context-parity rule
    fix: Record the fact in Constraints (lym heddle list shows every entry targeting claude-file or orch-prompt; the repository has no CLAUDE.md or AGENTS.md). Add to D12 (or a new Decision) that heddle's shipped-layer entries that apply to every agent also target codex-file, which is code in internal/heddleengine, and state that the repository-layer entries need the operator's lym heddle set. Replace the third Open risk with that one remaining operator step, naming the entries, and add an acceptance bullet that a Codex seat's codex-file render carries the shipped entries the chair's claude-file render carries.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 and Open risks (the [tui] settings)
    summary: D5 renders the [tui] settings Codex writes on first start without naming them, leaving the key set for the implementer to find out, though the twill spike records them
    fix: In D5, name the [tui] keys lym renders, screen_reader_detection_done and the model_availability_nux table, citing docs/research/codex-twill-spike-2026-10-24/report.md (Codex added both to a private config.toml). Rewrite the last Open risk to state only what stays unproven (that pre-setting them stops Codex writing on first start, seen in two spikes) and that D5's bound and D8's trust-prompt backstop cover a missed key.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D13 and D5 (auth.json link)
    summary: Whether a symlinked auth.json survives Codex's token refresh is untested (twill spike, Not tested), yet D13 rests on it; a temp-file rename would replace the link with a file and the two homes diverge silently
    fix: Research Codex 0.164.0's auth.json write path from its source (codex-rs login/auth storage) and record in D13 whether it writes in place or by rename, with the source path and version. If by rename, add to D5 that Prepare replaces a regular-file auth.json in the lym home with the link (after copying it back to the operator home if newer), and add an acceptance bullet for that case.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 bound and D5 (--dangerously-bypass-hook-trust with automatic folder trust)
    summary: D5 trusts every worktree and main checkout root automatically and D6 bypasses hook review, so hooks and settings committed in a repository's .codex/ run unreviewed and outside the sandbox, an edge the record neither states nor bounds
    fix: Add to D6's bound that automatic trust plus --dangerously-bypass-hook-trust runs any hook or setting committed under the repository's .codex/ without review and outside the sandbox (hook-delivery report case 5; spike Startup and trust), and what bounds it, such as lym only trusting worktrees of repositories the hub runs. Alternatively, state that Prepare refuses a worktree holding a committed .codex/config.toml, naming the file. Add the matching acceptance bullet.
  - id: F7
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks (Codex subagent fan before Step 4); support-log.md question ledger Q4
    summary: The entry defers forks to Step 4 yet names a forked Astra fan as the setup; whether D2 and D3 may let an operator turn on a Codex subagent fan before Step 4 is the parent's call
    fix: Parent to decide whether an opt-in Codex subagent fan (D2 and D3's assist fan keys) is allowed before board Step 4, or the assistant runs solo only in this task. On a no, drop the fan from D2 and D3 and their acceptance bullets; on a yes, record the answer in D2 and drop the Open risk and close Q4.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 bound and D6 (chair wait and first-turn retry)
    summary: The chair's wait bound and D6's retry claim do not hold on every path; start retries lengthen the wait, a dead lym process leaves no marker, and a positional prompt dropped while Codex stays alive is not a failed start shuttle retries
    fix: Restate D1's bound as the assistant's start (with its start retries) plus its timeout plus one await cap, and add that a lym process that dies before writing the marker leaves the chair waiting until its own review timeout or a resume's probe writes the marker. In D6, replace "which shuttle retries as any failed start" with what happens when Codex is alive but no SessionStart arrives, namely that the run ends at its timeout and the marker reads timeout.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 and D7 (mail channel to the assistant)
    summary: D1 says no channel to the assistant exists, but D7 makes its strand a pane-mail strand into which creel types any post, notice or reminder addressed to it
    fix: In D1, replace "and no channel to it exists" with the fact that creel would type a post addressed to the assistant's strand into its pane as for any pane-mail strand (D7), and state its bound, for example that no lym program addresses an assistant strand and its stencil tells it to ignore creel text.
  - id: F10
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D2 (assistant stencil's parent directive)
    summary: PATTERN-parent-directive requires the assistant's opening stencil to render the parent directive, whose parent variant tells it to post creel mail it cannot send, and D2 does not say which variant it renders
    fix: In D2, state that burler-template-assist-reviewer renders the parent and edit directives as PATTERN-parent-directive requires, the parent directive in its no-parent variant (internal/parentdirective with an empty parent name), since the assistant sends no message (PATTERN-agent-messaging), and that the edit directive's tool wording comes from an engine block (D11).
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks (skills sync) and support-log.md question ledger Q7
    summary: The record hands D9's load-by-path choice to the parent as an open risk while the ledger marks Q7 resolved; it is a choice the writer can make, and D9's rationale already settles it
    fix: Drop the first Open risk (skills sync versus load by path) and keep the departure from the entry's Step 2 wording as one clause in D9's rationale; leave Q7 resolved.
---

### [BLOCKING] The assistant's report path is outside the Codex sandbox

**Location:** `_lym/discussion/decision-record.md` D2, "The assistant's report and end marker sit at a path under the anchor's `.lym` … so the workspace sandbox covers them".

**Issue:** The anchor is the weft checkout: `codex-engine-weft/.lym-anchor`. The warp worktree's `.lym` and `_lym` are both links into it, `.lym -> /home/knatte/Code/loomyard-LYMHUB/codex-engine-weft/.lym`.
The spike says `workspace-write` makes only the workspace, `/tmp` and `$TMPDIR` writable (`docs/research/codex-spike-2026-10-24/report.md:283`), and that Codex keys a symlinked path by its resolved target (`report.md:10`).
The support log uses the same fact to rule out `_lym` (Turn 2), but `.lym` is the same kind of link.
So the assistant cannot write its report. The review-parse gate re-prompts until the timeout, the marker reads `timeout`, and every assisted round runs without the assistant.
Three lenses found this independently.

### [MEDIUM] A retry reads the previous attempt's marker or report

**Location:** D1 and D2, stale-marker removal and the stable per-round paths.

**Issue:** The review row runs a second attempt of a round under the same review path (`internal/shedadapters/burlerreview.go`).
The marker and report are derived from that path and the round. Only the copy beside the review is archived.
seatengine starts the chair first (`internal/seatengine/doc.go:51`), and D1 removes stale files only "before it starts the assistant".
`lym shuttle await` returns as soon as a path exists (`internal/shuttlecli/await.go:49`).
So a retry's chair can read the old marker and report. A crash between the chair's start and the assistant's start leaves the old report for the resume probe, which shuttle reads as all outputs present.

### [MEDIUM] Codex gets none of the repository instructions the chair gets

**Location:** D12, D6's `project_doc_fallback_filenames`, and the third Open risk.

**Issue:** The repository has neither `CLAUDE.md` nor `AGENTS.md`.
`lym heddle list` shows every entry, shipped (`lym-batching`, `lym-plan-edits-before-gate`) and repository (`nicknames`), targeting `claude-file` alone.
So `codex-file` renders nothing, and the `CLAUDE.md` fallback finds no file.
That defeats D12's stated purpose and the operator's rule of 2026-10-27 that a Codex reviewer gets the repository's agent instructions.
The Open risk calls this possible ("may") and "not code", but it is a fact checkable today, and the shipped entries live in `internal/heddleengine`.

### [MEDIUM] The `[tui]` keys are left for the implementer to find

**Location:** D5, "plus the `[tui]` settings Codex otherwise writes on its first start", and the last Open risk.

**Issue:** The render needs a concrete key set, but the record names none.
`docs/research/codex-twill-spike-2026-10-24/report.md:17` records the two keys Codex added: `screen_reader_detection_done` and a `model_availability_nux` table.
This is a fact the design rests on, so the record should state it rather than defer it.
If a key is missed, D5's bound and the trust backstop limit the damage, so the severity is MEDIUM, not BLOCKING.

### [MEDIUM] The `auth.json` link's survival across a token refresh is untested

**Location:** D13's rationale and D5's link maintenance.

**Issue:** D13 rules out a copy or a hard link because Codex rewrites the file, which means it assumes a symlink survives the rewrite.
The twill spike lists exactly that as untested (`docs/research/codex-twill-spike-2026-10-24/report.md:245`).
Codex's source answers whether it writes the file in place or replaces it by rename, at no run cost, which honours the operator's preference for desk studies.

### [MEDIUM] Automatic trust plus the hook-trust bypass runs repository `.codex/` hooks unreviewed

**Location:** D5 (trust every worktree and main checkout root) and D6's flag `--dangerously-bypass-hook-trust`.

**Issue:** Hook-delivery case 5 shows that a hook committed in the repository's `.codex/config.toml` runs with no prompt once the folder is trusted and the bypass flag is given.
The trust prompt warns that such hooks can run outside the sandbox (spike lines 52 and 71).
D6's bound covers only the sandbox and shell-guard hooks. It states neither this new edge nor what limits it, which the rubric's attack-surface question requires.

### [MEDIUM] The Codex subagent fan before Step 4 is the parent's call

**Location:** Open risks, second bullet; support log Q4 (open).

**Issue:** The entry defers forks to Step 4 but also names the forked Astra fan as the setup (operator, 2026-10-27).
The goal of 2026-11-03 sets aside only the replacement goal.
This is a contradiction inside the entry that only the parent can settle.
The record and the ledger agree that it is open. The record bounds the fan correctly in D2, solo by default, so the question is whether the fan may exist at all.

### [LOW] The chair's wait bound and D6's start retry are overstated

**Location:** D1's bound and D6's "which shuttle retries as any failed start".

**Issue:** Start retries happen before the assistant's timeout begins, so the wait can exceed "timeout plus one await cap".
If the lym process dies, no marker is written until a resume.
D6's retry claim fails when the positional prompt is dropped while Codex stays alive, for example after a backstop trust accept, because shuttle retries only a start that died.
Each of these paths still ends, at the chair's or the assistant's timeout, so the bound is misstated rather than missing.

### [LOW] D1's "no channel" contradicts D7's pane-mail strand

**Location:** D1, "no channel to it exists"; D7, `CreelHooks` false.

**Issue:** creel types injects, notices and reminders into every strand that has no creel hooks (`internal/creelengine/deliver.go:34-35`, `242-246`).
So a post addressed to the assistant reaches its pane, and the record states no bound for it.

### [LOW] The assistant stencil's parent directive is unstated

**Location:** D2, `burler-template-assist-reviewer`.

**Issue:** PATTERN-parent-directive (test) requires every spawned role's opening stencil to render the parent and edit directives.
The parent variant tells the agent to `lym creel post --parent`, which the assistant cannot do and must not do (PATTERN-agent-messaging; D1).
`internal/parentdirective` renders a no-parent variant for an empty parent name, but D2 does not say which variant the assistant gets.

### [LOW] The skills-sync risk is a writer's choice, recorded as open

**Location:** Open risks, first bullet; ledger Q7, "Resolved".

**Issue:** Step 2's "skills sync" is a feasibility step, not an operator decision. D9 departs from it on tested grounds: the `$` popup takes the next Enter, and skill discovery is untested.
The record hands the choice to the parent, while the ledger marks it resolved.
A choice the writer can make is never the parent's.

## Rejected

- Bound-breaker 3 and 4 and existing-paths F1 are not kept as separate findings: the probe that misreads a leftover report and the process-death wait are merged into F2 and F8, since each shares its root and its fix with them.
- Interactions F4 and walkthrough 2 duplicate F3. Interactions F2 and walkthrough 1 duplicate F1. Interactions F3 and existing-paths F3 duplicate F9. Walkthrough 4, interactions F1 and bound-breaker 2 duplicate F2. Each was merged, not rejected on merit.
- D2's fan bound is not a defect. D2 states what the fan lets through (no shell guard, no lym fork audit) and what limits it (the sandbox, the prompt, and the assistant's timeout from D3).
- The linked-worktree trust case resting on one sample is not a defect. D5 records both roots, and D8's trust-prompt backstop covers a wrong guess.

## Focus departures

None. The focus directives map to findings as follows:

- **Attack surface.** The fan's bound in D2 is stated. The stray-edit bound in D6 is stated, since it is the same exposure a Claude reviewer has. The hook-trust bypass has no stated bound (F6). The done-marker wait's bound is misstated (F8).
- **Deferred research.** The `[tui]` keys (F4) and the `auth.json` refresh (F5) are deferred facts. The one-sample trust case and the positional prompt are risks the design already bounds; the second also appears in F8.
- **Open risks against the parent.** The fan before Step 4 is the parent's call (F7). Skills sync is the writer's choice (F11).
- **Completeness before leanness.** No Decision or Open risk carries deliberation narrative that belongs in the support log. The support log's Turn 2 fact about `_lym` being a link out of the worktree is the one requirement the record lacks, and F1's fix carries it into Constraints.
- **Consistency.** D1 and D11 agree: a missing seat directive refuses the table, and a missing engine block only keeps the assistant from starting. Their acceptance bullets match the wording. D1 and D7 disagree (F9).
