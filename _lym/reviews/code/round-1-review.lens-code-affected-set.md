```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, amendment scope (`docs/overview.md` after the merge of the parent's drift-recipe edits).

**Per change, the set found:**
- The creel tree rows (`docs/overview.md:311-312`) now name `hook-read` and the mail flags with the tool-boundary claim. The matching code and docs agree: `internal/creelcli/doc.go:1` lists the same verbs, and `hookReadVerb = "hook-read"` is in `internal/creelcli/hookread.go:18`. Covered.
- The creel module bullet (`docs/overview.md:472-473`) now lists the hidden `hook-read` verb and adds the hook paragraph. Its claims match the code and the decision record. The `PostToolUse` hook is Claude-only, as `internal/shuttleengine/claudeengine/settings.go:70` and `settings.go:350` show. The flag-file test comes before `lym creel hook-read`, agent mail is included, and each message is delivered once, by the hook or by the idle pane path. Covered.
- The merged darn recipe sentence (`docs/overview.md:499`) and the regression-row sentence (`docs/overview.md:565`) are the parent's hunks. They do not touch the creel hunks. I found no conflict markers, duplicated lines or mangled sentences in either, and no stray residue near the creel lines. Covered.
- Other docs that list the creel verbs: a repo-wide markdown grep for the `release` verb list found only the two `docs/overview.md` lines above. The other hit is a research report that has nothing to do with the verb list. Covered.

**Findings:**

1. LOW, class `consistency`, `docs/overview.md:473`. The bullet says the strand gets its mail "in full at its next tool call". The decision record caps one claim at an 8 KiB budget (at least one message, the rest left unread with the flag standing). It also says the hook claims nothing while a pause covers the strand, and only the hold owner's mail under a hold.
   - These limits don't contradict the sentence, but "in full" reads as unconditional.
   - Fix: reword to "its mail, agent mail included, in full, up to a per-call size budget, at its next tool call". An unconditional "in full" is not contradicted by the record's hook decisions in any way that changes behaviour, so I rate it LOW.

No other member of the affected set is left wrong.

**Tool calls (2):**
1. `grep` of `hook-read`, `PostToolUse` and the creel lines in `docs/overview.md`: my context lacked the `internal/creelcli` verb wiring and the `claudeengine` settings lines.
2. `grep` for other docs that list the creel verbs, and for the `Drift-Await` and `Drift-Baseline` lines: my context lacked a repo-wide check for other docs needing `hook-read`.
