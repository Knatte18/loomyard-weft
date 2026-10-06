# Decision record — skills-in-one-turn

## Goal

A spawned role, and the orch after each cycle, loads every skill on its list in one model turn instead of one slash-command turn per skill.
A skill the session does not know never stalls a launch or a reload: it is skipped when that turn ends, or when it times out, with a Warn naming it.

## Scope

In:

- `internal/shuttleengine`: the `SkillLoader` capability and `Run`'s launch skill step (`loadSkillsThenPrompt`/`loadSkill`) move from per-skill to per-list.
- `internal/shuttleengine/claudeengine`: the one-message load sequence and a transcript check that classifies each requested skill.
- `internal/orchengine` watcher reload sequence and its `Session` seam, plus `internal/orchcli`'s adapter onto the runner.
- Issue #394: the unknown-skill stall, in shuttle's launch and the orch reload alike.
- `internal/loomcli/integration_driverbootstrap_test.go` and `internal/loomcli/smoke_driverstrand_test.go`: each shell stub's skill-load answer only (see Decision 14).
- Docs in the same commit: `shuttleengine/doc.go`, `claudeengine/doc.go`, `orchengine/doc.go` ("The reload sequence"), and the `Spec.Skills`/`Spec.SkillLoadTimeout` comments.

Out:

- Which skills each role loads: every per-role skill list stays as it is.
- When the orch watcher reloads (after clear, compaction, auto-compaction, at `start` and `--adopt`): unchanged.
- Re-typing skills into a Webster master after its auto-compaction: not added (see Decisions on compaction).
- Inlining skill bodies into prompt files.

## Decisions

1. **One typed message loads the whole list.**
   `claudeengine` types a single plain message (no leading `/`) that asks the model to load each listed skill through the Skill tool, all in this one turn, in list order, to do nothing else, and to reply only `ok`.
   Rationale: Claude Code treats only a leading `/<name>` as a command, so several skills cannot share a slash line; a Skill-tool load registers the skill as invoked, so its text arrives as user input and Claude Code re-injects it after a compaction.
2. **The message text is engine choreography, not a stencil.**
   It lives in `claudeengine` beside the `/clear` and `/compact` sequences; `shuttleengine` and `orchengine` pass only provider-neutral skill names.
   Rationale: the words "Skill tool" are Claude-specific (`PATTERN-shuttle-provider-seam`), and it is a fixed command line like `/compact`, not a producer prompt (`PATTERN-stencil-ownership` does not apply).
3. **The message is delivered through the verified send path** shuttle already uses for the prompt pointer, not a bare keystroke play.
   Rationale: it is a long plain-text line, which is what the verified path exists for.
4. **The load is verified from the transcript at the load turn's end.**
   At the turn end, the engine reads the transcript the Stop payload names and classifies every requested skill as one of:
   - **loaded** — a main-chain Skill tool call naming it, after the load message, with a successful result;
   - **unknown** — a skill listing was found in the transcript and the skill is absent from it, or its Skill call returned an error;
   - **missing** — none of the above (the model skipped or never reached it).
   Only entries after the load message count, so a Skill call from before a compaction or an earlier reload never confirms a new load.
   A transcript with no skill listing never classifies a skill `unknown` by absence (Decision 6 applies); a truncated listing can only misclassify a skill past the cut as `missing`, which costs one retry and never a silent skip.
   Rationale: the turn end alone proves nothing about which skills loaded; the transcript is where `claudeengine` already reads session facts (`ContextTokens`, `CompactedSince`).
5. **Unknown skills are skipped at once; missing skills are retried once.**
   An unknown skill is logged with `logger.Warn` naming the skill and cause `unknown` and is never retried.
   Missing skills get exactly one retry: one more message, same form, naming only the skills the first turn classified `missing`.
   The retry turn is classified with Decision 4's three classes; a skill that is `unknown` there is logged with cause `unknown`, and whatever is still `missing` is logged with cause `not loaded`, then skipped.
   Bound: at most two load turns per launch or reload, each bounded by the load timeout; a skip never fails or hangs the launch.
6. **An unreadable transcript degrades to "unverified", not to a retry.**
   No `transcript_path`, an unreadable file, no load message found, or no skill listing found: the turn end confirms the load, as today, with one Warn `skill load unverified` listing the skills.
   This applies to every load turn including the retry, whose skills are then confirmed unverified.
   Bound: this lets an unverified load through without a retry, never a failure; an unknown skill on such a turn goes undetected and appears only in that Warn's list.
   Retrying cannot help when the evidence is unreadable, and the transcript format is a Claude Code internal (the same degrade rule as `usage.go`).
7. **The timeout bounds one load turn, not one skill.**
   `Spec.SkillLoadTimeout` and `DefaultSkillLoadTimeout` keep their names and defaults and now bound each load turn (the first and the retry).
   A load turn that does not end in time has no turn end and no transcript read, so nothing confirms a skill: every skill the turn asked for is skipped with cause `timeout`, and a timed-out first turn is not retried.
   Bound: a skill that did load before the hang is logged as skipped but stays loaded in the session, and the prompt pointer is typed afterwards.
   The orch watcher keeps the handoff timeout as its per-step bound.
8. **The pane-text unknown-skill classifier is removed.**
   `SkillUnknown(capture, skill)` and its needles go from the `SkillLoader` capability, `Runner`, and the orch `Session` seam.
   Rationale: with no slash line typed, Claude Code's slash-command parser never answers; its `Unknown command: /<name>` notice (which the old needles never matched, the cause of #394's 600 s stall) cannot occur, and unknown skills now surface in the transcript at the turn end.
   Bound: on a turn whose transcript is unreadable an unknown skill is no longer detected and appears only in the `skill load unverified` Warn's list (Decision 6), with no stall.
9. **The `SkillLoader` capability becomes list-shaped.**
   It provides: the load sequence for a list of skills, the turn-end classification of a list against the transcript, and the default per-turn timeout.
   `Runner` exposes the same two per-list primitives for the orch watcher in place of `LoadSkill`/`SkillUnknown`.
   A spec naming skills on an engine without the capability is still refused at start; an empty list still changes nothing.
10. **Shuttle's events offset still ends past every load turn end** (first and retry), so `Wait` never reads a load turn as the run asking.
11. **The orch reload sequence becomes: skills step, optional retry step, pointer step.**
    The skills step types the one message when the idle probe passed on that tick; its turn end triggers the classification; unknown skills are skipped, missing ones move to a retry step typed under the same idle rule; then the pointer step runs as today.
    Persisted state carries the step, its first-typing time and events offset (as today) plus the skills still to retry, so a restarted watcher never re-sends a confirmed step, never retries twice, and never restarts a step's timeout.
    The retry step is its own persisted step value, distinct from every old per-skill index, and the move into it writes the step and the retry list as one state update; a watcher restarted before that write classifies the same transcript again to the same result.
    Rationale: the watcher's restart and idle-gating rules (`PATTERN-orch-pane-single-writer`) stay intact; only the step list shrinks.
12. **No migration for a reload in flight across an upgrade.**
    A persisted `reload_step` from the old per-skill indexing that falls past the new skills step, including an old index that would collide with the retry step's value, is read as the pointer step.
    Bound: the worst case is one reload that skips the remaining skills of that one cycle; the next cycle reloads them all.
13. **No extra reload for Webster or other spawned roles after compaction.**
    Rationale: Decision 1's Skill-tool load is assumed to be what Claude Code re-injects after a compaction, which would close the gap 70bf30ae8 listed ("A Webster master that Claude Code auto-compacts loses its skills"); the orch keeps reloading per Scope.
    Bound: the premise is unverified (Auto-mode assumptions); if it is wrong, a compacted Webster master or other spawned role runs without its skills, nothing detects it, and no fallback exists, as before this change.
14. **The loomcli stub-driver tests answer the new load message by ending the turn.**
    Both stubs stand in for `claude` and today answer a typed `/<skill>` line with `Unknown skill: <name>`, which only the removed pane classifier read.
    Each stub instead treats the first typed line as the skill-load message and ends that turn by appending a Stop event, with no `transcript_path`, to the run's events file, which it locates from its own launch arguments (the settings file sits in the run directory); the next typed line is the prompt pointer, as today.
    The load then confirms under Decision 6 at once, so `lyx loom start` stays within the tests' 30 s bound.
    Rationale: it exercises the real protocol end to end.
    Bound: the edit touches only each stub's skill-load answer and its comment, since `test-suite-prune-core` prunes `internal/loomcli` tests in parallel and the two runs must touch disjoint hunks.

## Constraints

- `PATTERN-shuttle-provider-seam`: the message text, the Skill-tool name, and every transcript shape (tool_use `name`/`input.skill`, tool_result success/error, the skill-listing attachment) stay in `claudeengine`.
- `PATTERN-role-skills-typed`: the spawning module still names skills on the launch spec and lyx types them; no stencil gains a load-skill instruction.
  The typed message is Go-owned engine choreography, not stencil text, so the entry's scan is unaffected and the entry text needs no change.
- `PATTERN-orch-pane-single-writer`: only the orch watcher types into the orch pane, idle-gated, on every reload step including the retry.
- `PATTERN-spawn-observability`: skips and unverified loads are logged via `internal/logger`.
- `PATTERN-test-speed` / `PATTERN-test-economy`: new coverage is untagged unit tests over fakes and transcript fixtures, extending the existing skill-load tests rather than adding parallel ones; no `llm` test.
- `PATTERN-comment-line-breaks` and the repo's semantic line breaks apply to every doc and comment touched.
- Docs land in the same commit (CLAUDE.md "Docs land in the same commit").

## Auto-mode assumptions

- The Skill tool call in a Claude Code transcript is an assistant `tool_use` entry with `name: "Skill"` and `input.skill: "<name>"`, answered by a `tool_result` whose record carries `toolUseResult.success`; the session's available skills appear in a `skill_listing` attachment. Verified against a live transcript of this session (Claude Code, 2026-10-06).
- A Skill-tool load is re-injected after a compaction, as the board entry states; not re-verified here.
- The model, asked plainly, loads several skills in one turn with parallel or sequential Skill calls and replies `ok`; Decision 5's retry covers the case where it does not.
- One retry is enough: a model that skips a listed skill twice in a row is not worth a third turn.

## Open risks

- The transcript format is a Claude Code internal and can change; Decision 6 bounds the damage to an unverified load, never a stall or failure.
- A model may answer with chatter despite "reply only ok"; that costs words, not turns, and nothing reads the reply.
- Re-injection after compaction is assumed, not verified (Auto-mode assumptions); if it is wrong, a compacted Webster master runs without its skills, as it does today.
- To check by hand after landing: launch a role whose list contains a nonexistent skill and confirm it is skipped at the first turn end with a Warn and no retry; run `lyx orch cycle` and confirm the reload takes one skill turn.

## Acceptance criteria

- A launch with N skills spends exactly one load turn before the prompt pointer when every skill loads.
- The typed load message is one line, starts with no `/`, and names every requested skill in list order.
- A skill absent from the skill listing, or whose Skill call errors, is skipped with a Warn naming it and cause `unknown`, with no retry and no wait past the load turn's end.
- A listed skill the model did not load is retried once in a message naming only the missing skills; still missing after that is skipped with a Warn.
- A Skill call from before the load message (earlier reload, pre-compaction) never confirms a skill.
- An unreadable or absent transcript, or a readable one with no skill listing, confirms the load at the turn end with one `skill load unverified` Warn and no retry, on the retry turn as on the first.
- A load turn that never ends is skipped at the load timeout, and the launch proceeds to the prompt.
- A pane that dies during a load turn is a died startup, as today.
- The run's events offset lies past every load turn end, including a retry's.
- An empty skill list changes nothing; a spec naming skills on an engine without the capability is refused at start.
- The orch reload sends one skills message, an optional retry, then the pointer, each only on an idle-probe tick; a watcher restarted mid-reload re-sends only an unconfirmed step, never a second retry, and keeps the step's original timeout.
- A persisted old-style `reload_step` beyond the skills step resumes at the pointer step.
- No `SkillUnknown` pane classifier or `/<skill>` load line remains in production code.
- The `tmux`-tier tests in `integration_driverbootstrap_test.go` and `smoke_driverstrand_test.go` pass with their stubs ending the load turn under the new protocol, with `lyx loom start` returning inside their existing 30 s bound and no wait on the skill-load timeout.

## Notes for the plan writer

- Real transcript shapes are visible in any recent `~/.claude/projects/<dir>/<session>.jsonl` with a Skill call; a trimmed copy makes a good `claudeengine/testdata` fixture next to the existing usage fixtures.
- `claudeengine/usage.go` already reads the transcript named by a Stop payload, backward, degrading on failure; the skill classification can follow the same reader.
- The orch watcher already holds the first turn end of a step (`seen.firstTurnEnd`), which is what the runner's classification primitive needs.
