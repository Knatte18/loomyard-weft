```yaml
lens: fix-consistency
outcome: reported
```

Task: run the fix-consistency check on the round-2 changes to `_lym/discussion/decision-record.md`.

1. **MEDIUM.** Changed text: D6's new paragraph, "The hub's worktree anchors reach both as a told list of absolute paths their caller resolves". It says how the hub's worktree records reach `RecordReaders` and the liveness engine, but not how they reach the other places that search "every known directory". Those are D2's fresh-launch search and resume search, the resume check (D2's new bullet, D6's first bullet) and D3 step 1. Each of them also needs the record directories. The fix made the told list explicit for two readers and left its twin, the launch-time search inside `Prepare`/`OpenLaunch`/`CheckResume`, with no stated input. Evidence: D2 "first searches every known directory (D6)"; D2 resume bullet "searched among every known directory (D6)"; D6 "reach both". `CheckResume(sessionID, workdir)` (`internal/shuttleengine/engine.go:336`) is told only a session id and a workdir.

2. **MEDIUM.** Changed text: the new D4 bullet, "`ScrubbedEnv`, also names `CLAUDE_CONFIG_DIR` … An operator's shell value of `CLAUDE_CONFIG_DIR` then reaches no lym pane either". Nothing limits the scrub to a hub. Every reed server spawn applies it (`shuttleengines.EnvScrub`, `internal/shuttleengines/shuttleengines.go:157-162`; `internal/reedengine/lifecycle.go` `e.scrub.Apply(os.Environ())`). That changes standalone runs too, which contradicts two unchanged lines:
   - Scope Out: "Standalone runs, outside a hub: they have no machine layer, load no accounts and keep today's behaviour".
   - D2's first bullet: "With `claude.accounts` empty, as it always is outside a hub…".

   D1's acceptance criterion with an empty list ("neither set nor unset") still passes, but panes no longer inherit an operator's shell value. Nothing in Scope or D2 states that change.

3. **LOW.** Changed text: D5, "The command runs a lym verb with two arguments: that account's name and its config directory". Several sibling lines still describe a name-only verb:
   - D5's first paragraph: "passes the entry's current name, never the name stored in the record". It should name the directory too.
   - D5 criterion: "the status line verb, given an account name and input with both windows, writes…". It should give the directory argument too.
   - D5 criterion: "…carrying the status line command with that entry's current name…". It does not mention the directory argument.

4. **LOW.** Changed text: D6's third bullet. `LocateSessions` now reads `projects/` of "every known directory, … never through the worktree's record", and the default `~/.claude` is one of them. The unchanged D6 criterion "the engine's transcript reader, resume check and plugin lookup for a worktree recorded on a fake directory find the files in that directory and not under `~/.claude`" now contradicts it, if "transcript reader" means `LocateSessions`. The first bullet narrowed to "the live session's transcript reads", but the criterion was not narrowed with it.

5. **LOW.** Changed text: D1, "read from each tracked layer's own document…", which replaces the provenance mechanism. The support log still says "bound-breaker 6, layer provenance for the tracked-layer refusal: fixed; walkthrough found `configengine` reports provenance, and D1 names it" (support-log Turn 3). D1 no longer names provenance. The log also has no entry for this review round ("_No rounds yet._").

6. **LOW.** Changed text: D3's new seeding paragraph. Its counter file now also holds each name's weight and "the names listed at the last pick, rewritten at every pick", and raised counts take part in the comparison. Step 5 still says only "It increments the taken account's count and writes the worktree's record". It does not say whether the raised counts, weights and listing are written. The paragraph's "rewritten at every pick" covers the listing but not the raised counts.

No further tool calls made.
