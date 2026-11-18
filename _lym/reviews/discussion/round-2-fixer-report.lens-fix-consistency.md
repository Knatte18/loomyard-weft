```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling text, criteria and cross-references it touches (fix-consistency lens).

Changed text checked:
- D2 (packages/llm_driving bounds, the Preflight live-tier refusal, the rationale)
- D4 (the binaries-started rule, the `--copies` paragraph)
- the D5 pairteardown reap
- D6 (the auto_accept bound)
- the D8 rationale
- D9 (the classification and the run_settings list)
- the removed treadle Open risks bullet
- the new and changed acceptance criteria
- support-log Q13

**[MEDIUM] D5's new pairteardown sentences: the route is called "the reap" but runs `clear`, which D5 defines as less than the reap.**
- Changed text: "`internal/pairteardown` also runs it before it removes a worktree whose sandbox root exists, by spawning `lym harden sandbox clear` in that worktree…"
- Evidence: unchanged D5 text defines the verb narrowly: "`lym harden sandbox clear` stops each hub's reed server and removes the root's hubs."
- The reap is defined separately. It also ends each hub's recorded processes through `runproc.End` and kills unrecorded processes found by the `/proc` scan.
- So the pairteardown route leaves the sandbox hubs' recorded dobbies alive, which is the F19 premise ("reed servers and dobbies … survive").
- The changed text and the unchanged definition of `clear` disagree. Either `clear` must run the full reap, or the new text must say what it leaves.

**[LOW] Scope "In:" doesn't name the two new cross-module behaviours.**
- Changed text: the D5 pairteardown route and D6's "a Harden-Bouncer never auto-accepts a round whose live report is red".
- Evidence: Scope "In:" lists "Go's reap and stray record around each round row" and "what the Bouncer's facts carry for a Hardener round".
- Neither covers a change to `internal/pairteardown`, or a change to the shared `Bouncer` engine's auto-accept path (`internal/shedadapters/bouncer.go`).
- The Scope twin was left as it was.

**[LOW] Support-log Turn 3 still says the Open risks line holds a parent question.**
- Changed text: the removed Open risks bullet and the resolved Q13.
- Evidence: support-log.md line 46 still reads "board-scope (MEDIUM, decision): the operator note lists treadle to build on. Kept D8 and turned the Open risks line into a question for the parent."
- That bullet no longer exists, and Q13 now reads resolved.
- The log line describes a past turn, but it doesn't say the operator entry later settled the question, so a reader can still look for a question that isn't there.

No further tool calls made.
