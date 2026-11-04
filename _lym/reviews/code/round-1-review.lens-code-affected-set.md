```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set I found and whether the diff covers it.**

1. **Matcher gains `Bash`** (`batchingNudgeMatcher`)
   - Set: `settings.go:432` uses the constant. `settings_test.go` and `TestPrepare_BatchingNudgeHook` compare against the constant. `claudeengine/doc.go:163`, `docs/overview.md` and the `template.yaml` `lookup_chain` comment all name the matcher or tool list.
   - Covered: the `doc.go` matcher text, the `overview.md` shuttle paragraph and the `template.yaml` `lookup_chain` comment are updated. `settings.go:8` and `settings.go:375` say "a tool a breach is made of", which stays true.

2. **Bash lookups count as lookups**
   - Set: `kindOf`, `judgeBatching`, the `doc.go` chain definition, the `template.yaml` comment, `overview.md` and `breaches.sh`.
   - Covered: all of them are updated.

3. **Subagents and forks are judged on their own** (`agent_id` reaches the judgment)
   - Set: the `doc.go` paragraph, the `overview.md` text, the `BatchingNudger` interface doc, the `hook-batching` verb's help text, the `template.yaml` `max_nudges` and `context_tokens` comments, `BatchingNudgeConfig.MaxNudges`, and the research `README.md`.
   - Covered: the `doc.go` paragraph, the `overview.md` text, and the `BatchingNudgeConfig.MaxNudges` doc, which now says "and each of its subagents and forks". The `engine.go:269` interface doc is generic and still true.
   - Not covered:
     - `template.yaml:19` and `hookbatching.go:26-33`, findings 1 and 2.
     - `template.yaml:15`, finding 3.
   - `README.md` line 3 is a measurement description, not a rule, and is unaffected.

4. **`latestReading` and `tailMessages` take a `sidechain` switch, and the `subagents` constants are named**
   - Set: callers in `usage.go:62` (passes `false`) and `batchingnudge.go:245,249`. Every hard-coded `"subagents"` literal in the package is replaced: `activity.go` (two), `audit.go`, `locateSessions`, `usage.go`.
   - Covered. The grep shows no literal left.

5. **State file gains an `agents` map and a per-agent count**
   - Set: `writeBatchingNudgeState`, `TestPrepare_BatchingNudgeHook`, `TestBatchingNudge`. Prepare's seed uses `omitempty`, so the seeded shape is unchanged.
   - Covered.

6. **Research scripts and the report** (Wanted 3)
   - Set: `breaches.sh`, `events.sh`, `events.jq`, `run-all.sh`, `report.md`, `README.md`.
   - Covered: only the `breaches.sh` comment changed. `events.sh:39` already reads `subagents/*.jsonl`, so a fork's nudge attachment is still found.
   - `report.md:184` ("The hook skips every call from a subagent or fork") is now false. It is a dated measurement report, which the prose skill exempts, so I raise no finding.

7. **PATTERN entries and declared changes**
   - No PATTERN entry is declared, changed or touched. The commit messages carry no `Pattern-Departure:` trailer.
   - `PATTERN-test-speed` and its test-economy rules are the lens-3 tests fork's.
   - I found nothing for this lens.

8. **Commit messages and the change description**
   - The first commit's message matches the diff. The two later commits only reflow comments.
   - The summary's claims about the matcher, constants, `breaches.sh` and `events.sh` match what I read.
   - Nothing outside the diff.

**Findings**

1. **LOW, consistency**: `template.yaml:19` is stale.
   - It says `max_nudges` is "the most nudges one run's session gets". Each subagent and fork now has its own cap, as the diff's own `BatchingNudgeConfig.MaxNudges` doc says (`batchingnudge.go:43`).
   - Evidence: `template.yaml:19`; `batchingnudge.go:43`, `BatchingNudge` at `batchingnudge.go:237-238`.

2. **LOW, consistency**: `hookbatching.go:26-33` is stale.
   - The verb's help text says it judges "the call and the session's recent messages" and prints nothing "past its nudge cap". Judgment is now per chain, from the calling agent's own transcript.
   - Evidence: `hookbatching.go:26-33`; `batchingnudge.go:206-210`.

3. **NIT, consistency**: `template.yaml:15` is stale on one point.
   - `context_tokens` is described as "the context size ... below which the hook stays silent". For an agent's call it is now that agent's own context. This is the weakest of the three, since "context" still reads naturally.
   - Evidence: `template.yaml:15`; `batchingnudge.go:245-247`.

**Tool calls**
1. A grep of `shuttleengines/doc.go`, `claudeengine.go`, `config.go`, `settings.go`, `hookbatching_test.go`, `shuttleengines_test.go` and `shuttleengine/*.go` for other descriptions of the hook, plus the `overview.md` diff text. My context held only a truncated `overview.md` diff.
2. Read `engine.go:262-282`, the `template.yaml` `max_nudges` and `context_tokens` lines, `report.md:180-186` and the `README.md` batching lines. My context lacked the `BatchingNudger` interface doc and these comments.
