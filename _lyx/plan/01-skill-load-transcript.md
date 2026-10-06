# Card 1 — skill-load-transcript

**Create:**
- `internal/shuttleengine#SkillLoadReport`
- `internal/shuttleengine/claudeengine#Claude.SkillLoadMessage`
- `internal/shuttleengine/claudeengine#Claude.ClassifySkillLoad`
- `internal/shuttleengine/claudeengine/skillload.go`
- `internal/shuttleengine/claudeengine/skillload_test.go`
- `internal/shuttleengine/claudeengine/testdata/skillload-mixed.jsonl`
- `internal/shuttleengine/claudeengine/testdata/skillload-no-listing.jsonl`

**Edit:**
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine#Event`
- `internal/shuttleengine/claudeengine#readBackward`
- `internal/shuttleengine/claudeengine#initialReadChunk`

**Intent:**
Add the provider-neutral result type and Claude's one-message load and transcript check, additively, so the next cards can switch callers onto them (Decisions 1, 2, 4, 6).
Nothing calls the new methods yet, and the old `/<skill>` path keeps working until card 4.

`SkillLoadReport` in `internal/shuttleengine/engine.go`, beside `SkillLoader`, is the turn-end classification of one load turn: `Verified bool` plus `Loaded`, `Unknown` and `Missing []string`, each in request order.
`Verified` false means the evidence was unreadable (Decision 6); the three lists are then empty and the caller treats every requested skill as confirmed unverified.
Its comment states the three classes and the unverified case in provider-neutral words.

In the new `claudeengine/skillload.go`:

- `SkillLoadMessage(skills)` returns one line with no leading `/` that asks the model to load each listed skill through the Skill tool, all in this one turn, in list order, to do nothing else, and to reply only `ok`, naming every skill in order.
  It is engine choreography beside `/clear` and `/compact`, not a stencil (Decision 2).
- `ClassifySkillLoad(turnEnd, skills)` reads the transcript the Stop payload's `transcript_path` names, walking it backward through `readBackward` like `usage.go`, and degrades to `Verified: false` on a missing `transcript_path`, an unreadable file, no main-chain user entry whose text equals `SkillLoadMessage(skills)`, or no `skill_listing` attachment.
  The latest such user entry is the load message; only main-chain entries after it count, so a Skill call from before a compaction or an earlier reload never confirms a skill.
  A skill is `Loaded` when a main-chain assistant `tool_use` with `name: "Skill"` and `input.skill` naming it is answered by a `tool_result` whose record carries `toolUseResult.success: true`;
  `Unknown` when its Skill call's result is an error, or when it is absent from the latest `skill_listing` attachment's `- <name>:` lines;
  `Missing` otherwise.
  Sidechain entries and malformed lines are skipped.
  All transcript shapes stay in this file (`PATTERN-shuttle-provider-seam`).

The fixtures are trimmed copies of a real Claude Code transcript.
`skillload-mixed.jsonl` holds a `skill_listing` attachment naming `a`, `b` and `c`, a successful Skill call for `a` before the load message, the load message for `a b c ghost err`, then successful calls for `a` and `b`, an error result for `err`, a sidechain Skill call for `c`, and a Stop-time assistant `ok`.
`skillload-no-listing.jsonl` holds the same load turn with no `skill_listing` attachment.

`skillload_test.go` is untagged, `t.Parallel`, and pins at the methods' surface:
one table over `ClassifySkillLoad` with rows for the mixed fixture (`a`, `b` loaded; `ghost` and `err` unknown; `c` missing despite its pre-message and sidechain calls), the no-listing fixture (unverified, never unknown by absence), a transcript whose load message names a different list (unverified), an unreadable path, and a Stop payload with no `transcript_path`;
and one test that `SkillLoadMessage` is a single line, starts with no `/`, and names every skill in list order.

`claudeengine/doc.go` gains a paragraph beside its skill-loading one describing the one typed message and the turn-end transcript check with its unverified degrade, and says the per-skill `/<skill>` path remains until it is replaced; the old paragraph stays true until card 4 removes it.

**ImpactSummary:** Additive: one new shuttleengine type and two new Claude methods with no caller yet; the claudeengine doc paragraph is rewritten.

**Commit:** `1: claudeengine: one-message skill load and transcript classification`
