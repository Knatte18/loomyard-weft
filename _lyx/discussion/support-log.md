# Support log — skills-in-one-turn

## Interview

Autonomous mode: no operator answers, so every turn below is a self-pick recorded in the Question ledger.

1. Read board entry 130 (body plus imported issue #394) and the board list.
   Related notes: `orch-watcher-stale-reload` (134) touches the same watcher but a different trigger; `strand-colors` and `agent-permission-setup` touch spawning, not skills. No overlap to merge.
2. Explored `shuttleengine` (`SkillLoader`, `Run.loadSkillsThenPrompt`/`loadSkill`), `claudeengine/session.go`, `orchengine` watcher reload sequence and `orchcli` skill lists, and commit 70bf30ae8.
   Found #394's root cause: the needles are `unknownskill:<name>` and `unknownslashcommand:<name>`, but Claude Code prints `Unknown command: /<name>`, which normalizes to `unknowncommand:/<name>` and never matches.
3. Checked a live transcript of this session: a Skill call is a `tool_use` with `name: "Skill"`, `input.skill`, and a `tool_result` with `toolUseResult.success`; a `skill_listing` attachment lists the session's skills.
   This made a transcript check feasible and also gives an "unknown" signal without an extra turn.
4. Settled: one message per load, transcript classification at the turn end, unknown skipped without retry, missing retried once, unreadable transcript degrades to unverified.
5. Settled the orch side: the per-skill steps collapse into a skills step, an optional retry step, and the pointer; restart rules carry over.
6. Challenged the problem: is a separate load turn needed at all, or can the skills ride in the prompt-pointer message? Kept the separate turn (see Rejected alternatives).

## Rejected alternatives

- **Inline skill bodies into the prompt file.** Rejected on the board already: arrives as a Read result, not user input, and a compaction summarizes it away.
- **Put the skill-load request in the prompt-pointer message itself.** Saves the one remaining turn, but a missing skill would be found only after the role's whole task turn, too late to retry, and the events-offset rule that keeps load turns out of `Wait` would no longer separate load from work.
- **Keep the slash-command turns and fix only the unknown-command needle.** Fixes #394 but not the turn-per-skill cost the task exists to remove.
- **Pre-flight the skill listing before typing anything.** The transcript path is only known from a Stop payload, and before the first turn there is none; the post-turn check gets the same answer for free.
- **Retry unknown skills too / retry more than once.** An unknown skill cannot load by asking again; a model that ignores a listed skill twice is not worth a third turn.
- **Treat an unreadable transcript as all-missing and retry.** Retrying cannot produce readable evidence; it only adds a turn.
- **Keep the pane-text `SkillUnknown` classifier as a fallback.** With no slash line typed, the notice it looks for cannot appear; it would be dead code.
- **Migrate an in-flight old `reload_step`.** The case needs a deploy mid-reload; the bounded worst case (one cycle without some skills) does not justify migration code.
- **A short `SkillLoadTimeout` for the loomcli stub-driver tests (parent review F1).** It would make every launch in those tests wait out a timeout instead of exercising the turn-end path, and the driver spec is built in production code, so a test-only timeout would need a new seam.
- **Make the load message a stencil.** It is fixed provider choreography like `/compact`, and the Skill-tool wording is Claude-specific, so it belongs behind the provider seam.

## Review rounds

_No rounds yet._

### Parent review — round 1

Verdict BLOCKING; one finding.

- **F1 [BLOCKING] loomcli stub-driver tests depend on the removed pane classifier** — fixed.
  Added both test files to Scope, Decision 14 (each stub ends the load turn by appending a Stop event to the run's events file, confirmed as unverified under Decision 6; edit limited to the stub's skill-load answer so it stays disjoint from `test-suite-prune-core`), and an acceptance criterion that both `tmux`-tier tests pass within their 30 s bound with no skill-load timeout wait.
  Chose the Stop-event answer over a short `SkillLoadTimeout`, which is recorded under Rejected alternatives.

## Question ledger

- **Q1 (auto-pick): How is a load verified?** Transcript classification at the load turn's end (loaded / unknown / missing). Rationale: the turn end alone says nothing per skill; `claudeengine` already reads the transcript for other session facts.
- **Q2 (auto-pick): Retry policy.** Unknown: skip at once with Warn. Missing: one retry naming only the missing skills. Rationale: bounded at two load turns, and the board asks for "retries a missing one".
- **Q3 (auto-pick): Unreadable transcript.** Confirm on turn end, one Warn, no retry. Rationale: matches `usage.go`'s degrade rule; never fail or stall on a Claude Code internal format.
- **Q4 (auto-pick): Timeout semantics.** The existing per-skill timeout becomes per load turn, same names and defaults. Rationale: one turn now loads the list; renaming fields buys nothing.
- **Q5 (auto-pick): Where does the message text live?** `claudeengine`, beside `/clear` and `/compact`. Rationale: `PATTERN-shuttle-provider-seam`.
- **Q6 (auto-pick): Does `PATTERN-role-skills-typed` need an edit?** No. lyx still types the load and no stencil asks for a skill; the entry's wording holds.
- **Q7 (auto-pick): Should Webster gain a post-compaction reload?** No. The Skill-tool load is the kind Claude Code re-injects after compaction, which closes that gap without new code.
- **Q8 (auto-pick): Orch state across an upgrade mid-reload.** No migration; an out-of-range old step reads as the pointer step.
- **Q9 (auto-pick): Separate load turn or merge into the prompt pointer?** Separate, so a missing skill can be retried before the work starts.
- **Q10 (resolved by reading): Cause of #394.** Needle mismatch, described in Interview turn 2; removed with the classifier.
