```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens, checking whether each guard, refusal, branch or test row the plan adds can be reached from a real entry point.

**Per guard, reachable or not**

- **Card 2:** the tracked-layer refusal of `accounts` and `account_pins.*` is reachable. configengine refuses no machine-category key from the default or worktree layer (`internal/configengine/doc.go:46-58,128-142`), so the hub file and worktree file rows reach it, also with the machine file set. The `validateAccounts` rows are reachable, since `ValidateMachine` checks only units and shape.
- **Card 3:** the "failed prime read is taken as prime" branch is reachable in principle, but no test reaches it, and the card says so.
- **Card 4:** an undecodable account or usage file reading as the zero record is reachable.
- **Card 5:** the no-op cases, the failed `Names` refusal, the later-`Picked` keep and no candidate are reachable through the seeded state and fakes. The pick after `top.Add` is reachable only on the first entry: a re-entered row takes the `complete` branch, and that branch never picks.
- **Card 6:** the config-load refusal before any artifact is reachable.
- **Card 8:** a carry failure refusing the resume, and a held refusal carrying `TranscriptCopied`, are reachable.
- **Card 9:** the unlisted-name refusal, the retired or unlisted `set` refusal, the hub-pin refusal and the outside-hub refusal (under `t.Chdir`) are reachable.
- **Card 11:** `RelaunchStrand` refusing a strand that is not held or not tracked is reachable. `EndStrandProcess` failing after the mark is reachable only through the shuttle fake.
- **Card 12:**
  - The `unfinished`, `runIsLive`, `StopStepRuns`, `dispositionCandidate` and activity/session-state rows are reachable.
  - The `sweepOrphans` "keep a relaunching record within its bound" guard is not reachable (finding 4).
  - `strandRemovable` reading a held strand as removable recurs at a site the card does not name (finding 5).
- **Card 13:** the hub pin, fewer than two accounts, an unlisted launched directory, "already resolves elsewhere" and no eligible account are reachable. `FindRun` by the orch GUID works because the orch starts through `runner.Start` (`internal/orchcli/start.go:198`).
- **Card 14:**
  - The terminal-at-mark case, the `ErrNoTranscript` fresh fallback (a transcript in no known directory), any other refusal ending died, and `EndStrandProcess` failing over a live or dead strand are reachable, the last two through fakes.
  - The usage-limit switch in `Wait` is reachable only for a limit read after the start finished; a limit on the start path is not guarded (finding 1).
  - The row "a limit hit during the skill-load turn keeps the deferred prompt line" is not reachable (finding 1).
- **Card 15:** past the bound, a not-live strand ending died and a live strand clearing the mark and attaching are both reachable; the held strand stays tracked in `reed.json`, so `dispositionCandidate` sees it. The `checkLivenessTick` within-bound case is reachable.
- **Card 16:** the `retryAPIError` case for a copied resume is reachable. The start-path case at `loadSkillsThenPrompt` is not reachable at the site the card edits (finding 2).
- **Card 17:** the watcher's usage-limit switch branch is reachable, since the watcher reads the newest turn end at idle (`internal/orchengine/watcher.go:305-321`). The `RefusedResume` branch is reachable on an idle turn end.

**Findings**

1. **BLOCKING**, design — card 14 (`internal/shuttleengine#Run.Wait`) and card 13.
   - A usage limit on the skill-load turn never reaches `Wait`. `launch` runs `loadSkillsThenPrompt` synchronously (`internal/shuttleengine/run.go:784`), and `readLoadTurn` reads a usage-limited turn end as an API error (`internal/shuttleengine/loadturn.go:232`; `newestTurnEndError` sets `apiError` for `rate_limit`, `internal/shuttleengine/claudeengine/activity.go:169-189`).
   - The load hold then retypes the load on the limited account until `loadHoldMaxAPIErrors` (8) and ends `OutcomeAPIError` (`internal/shuttleengine/loadturn.go:154,294-318`).
   - So on that path no `blocked_until` is recorded (D5) and no switch runs (D7).
   - Card 14's row "a limit hit during the skill-load turn keeps the deferred prompt line and types it after the relaunch" describes a state `Wait` cannot see.
   - Fix: carry the limit check into `loadSkillTurn` or `readLoadTurn`, or drop the row and record that load-turn limits are handled as today.
2. **MEDIUM**, design — card 16 (`internal/shuttleengine#Run.loadSkillsThenPrompt`).
   - The start-path refused resume cannot be decided at the site the card edits. The first API-error turn end of the load turn is consumed inside `loadSkillTurn` and `readLoadTurn`, which hold and retype (`internal/shuttleengine/loadturn.go:136-241`).
   - `loadSkillsThenPrompt` sees only a passed load or the capped error after 8 errors, which `abandonLoad` handles at `internal/shuttleengine/loadturn.go:294`.
   - So the row "a copied resume whose skill-load turn ends on an API error starts fresh from the start path" cannot be reached at the first turn end as the row states.
   - Fix: list and edit `loadSkillTurn` or `readLoadTurn` (`loadturn.go`) to break out on the first API error of a copied resume.
3. **LOW**, design — card 5 (`internal/battencli#battenCLI.wire`).
   - The pick runs only after `top.Add` succeeds. A row re-entered after a crash between Add and the pick takes the `complete` branch, which returns `reconcileTaskPair` without picking (`internal/battencli/wire.go:1136-1145`).
   - That worktree stays on today's directory. D3 says the pick runs when the worktree is made.
4. **LOW**, design — card 12 (`internal/shuttleengine#sweepOrphans`).
   - The guard that keeps a relaunching record within its bound "whatever its strand" guards a state that cannot occur. The sweep's GUID set holds every strand in `reed.json`, live or not (`internal/shuttleengine/run.go:1197-1200`).
   - Card 11 keeps the held strand's record there throughout the relaunch, and every path that removes it first makes the record terminal (the stop) or finalizes it (the died end).
   - The row passes only on a seeded absent GUID. The guard is harmless, since D7 asks for it, but no real entry point reaches it.
5. **LOW**, consistency — card 12 (`internal/shuttleengine#strandRemovable`).
   - Reading a held strand as removable also changes `removeSupersededStrands` (`internal/shuttleengine/attach.go:233-245`) and `StopStepRuns`' untied branch (`internal/shuttleengine/stepstop.go:98-101`). Neither is named in the card.
   - A relaunching run that is a superseded attach candidate will now have its held strand stopped where today it is skipped. The card should state whether that is intended.

Tool calls:
1. Read `loadSkillsThenPrompt` in `run.go`, to see whether the load turn runs before `Wait`; the manifest lacked it.
2. Read `abandonLoad` and the `PromptLine` uses in `run.go`, to see how a load-turn error ends the start.
3. Scanned `loadturn.go`, to see how the load hold treats an API-error or usage-limit turn end.
4. Read `sweepOrphansOpportunistic` and the claude `activity.go` classification, to get the sweep's GUID set and to confirm a usage limit reads as an API error.
5. Read the orch `Watcher.tick` turn-end handling, to confirm the card 17 branches can be reached.
