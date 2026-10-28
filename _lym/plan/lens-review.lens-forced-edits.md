```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. Five findings. Each is a doc, test reference or comment that a card's change makes wrong, but that no card lists.

1. LOW — Card 16 (D9), shuttle row of the overview. `docs/overview.md:409` still says the engine announces each deny "(`--append-system-prompt`, on launch and resume)". Card 16 replaces that flag with `--append-system-prompt-file` and the trust sentence. Card 16 does not list `docs/overview.md`. Card 17 lists it, but its Intent covers only the creel row and the module-table lines. The shuttle row stays wrong.

2. LOW — Card 17 (D6/D11), the sandbox creel suite. Card 17 covers only part of `tools/sandbox/SANDBOX-CREEL-SUITE.md`: it rewrites S9 alone. These sections still describe reminders typed into a Claude strand's pane, which cards 9 and 11 remove for any strand that is not pane-mail:
   - the intro and judging notes at `:6` ("mail reminders … typed into a strand's pane") and `:49` ("C2's single reminder in the scrollback");
   - the busy-turn scenario at `:152-157` ("confirm its reminder is typed after the turn");
   - the backoff scenario at `:165-167` ("the strand answers reminders").

   Those scenarios would now fail or test nothing.

3. LOW — Card 12 (D1/D3), capability docs outside claudeengine. `internal/shuttleengines/doc.go:21` describes `ToolBoundaryRenderer` as "one registered engine's tool-boundary context renderer". `internal/shuttleengine/doc.go:27` lists "tool-boundary context rendering" among the optional capabilities. After card 12 the capability parses hook input, renders the visible block and gives the rewake output. Card 12 lists only `internal/shuttleengine/claudeengine/doc.go`. The doc comment on `internal/shuttleengines#ToolBoundaryRenderer` ("as its tool-boundary context renderer") is just as stale and is not edited either.

4. LOW — Card 10 (D4), the background file for `PATTERN-daemon-wakeups`. `pattern/PATTERN-daemon-wakeups.md:27` lists creel's waiting verbs as "`post --wait-reply`, `inject --wait` and `read --wait`", which wait on their connection and reconnect from one second. Card 10 adds a new unbounded waiting verb, `hook-wait`, which blocks for up to 86400 s and reconnects the same way. No card adds it to that list, so the background file's list of conforming waits is incomplete.

5. LOW — consistency between cards 2 and 14. Card 2's Intent says "Claude's `Prepare` sets the field in card 13". The card that sets `Launch.CreelHooks` in `Claude.Prepare` is card 14, so the cross-reference names the wrong card.

Tool calls beyond my context:
- Grep for callers of `ToolBoundaryContext`, `ClaimAtBoundary`, `buildLaunchCmd`/`buildResumeCmd`, test uses of `IdleProbe{`, `hook-read` registrations and the `waitKind` definitions. My context did not have the full caller sets.
- awk and `lym quarry resolve` to map those callers to their test functions and check the glyphs the cards name. My context did not have the enclosing test names or quarry's answers.
- `lym quarry resolve` again for the raw answers on `fakeEngine`, `Waiter.Cancel`, the `decisionFake` helpers, `TestToolBoundaryRenderer`, `InterruptGuard.RunningTurn` and `init`. The first call's filter dropped its output.
- `docs/overview.md:409`, the sections of `SANDBOX-CREEL-SUITE.md`, the shuttleengine and shuttleengines doc mentions, and the orchengine test. My context had only the matching lines, not what they say.
- The `[creel]` and reply lines in the parent, webster and soft-handoff stencils, the build tags of the creelengine test files, and the docs that mention `wait-reply`. My context did not have the stencil text or the tags.
- The creel lines of `pattern/PATTERN-daemon-wakeups.md`. My context did not have that background file.
