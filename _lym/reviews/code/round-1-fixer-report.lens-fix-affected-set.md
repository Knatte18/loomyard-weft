```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for F1, F2, F5, F6 and F7 and check the fix covers every member.

Findings:

- **LOW** — `internal/shuttleengine/send.go:592`, the `ErrCommandTakenAsPrompt` doc comment.
  - It still says the error marks a command "whose paste started a turn".
  - F2 narrowed the error to a turn start whose instruction equals the pasted text. The `confirmPasted` and `sendContext` comments and `doc.go` were updated, but this sibling was not.
  - A reader of the error's doc would still think any turn start past the paste raises it.

Findings with no gap:

- **F1** — `startsNoTurn` is set at one site only, `sendSequence` at `send.go:701`. Its only consumer, `pasteVerified` at `send.go:623`, passes `reader` to `clearBoxBeforeCommand`. The `retryLanded` and `confirmPasted` uses of `startsNoTurn` were handled. No missed caller.
- **F2** — the sole `startsNoTurn` branch in `confirmPasted` now uses `promptTurnStartedSince`. `retryLanded` at `send.go:736` already excludes `startsNoTurn`. No other `turnStartedSince` consumer needs the instruction match.
- **F5** — the creel docs that list the command inject's outcomes are `creelengine/doc.go:412` and `creelcli/doc.go:149`. Both got the new sentence. `creelengine/doc.go:30` and `:395` describe typed lines, not commands.
- **F6** — the only other site of the "launch and resume" TTL set is `claudeengine/doc.go:96`.
  - It says "The launch and resume lines both carry `CLAUDE_CODE_PROMPT_CACHE_TTL`".
  - `claudeengine/doc.go:103` already covers `OpenLaunch` resolving and validating the TTL. The sentence at line 96 is true as written, so I count no gap.
- **F7** — the only places that say the TTL is "validated when a run starts" are `template.yaml:3` and `:4`, and both are updated. The `keep_warm_cap` lines say "validated when a run starts and on lym config claude", which is a different setting and out of scope.
- **F3** and **F4** — each adds rows to an existing table (one test, one helper table). The set premise is covered.

Unchecked: nothing. No further `lym` or test run was needed.

Calls made:
1. Grep for the `ErrNotLanded` / `ErrBusy` command-inject docs (F5), to enumerate the doc sites.
2. Grep for "launch and resume" across `.md`, `.go` and `.yaml` (F6), because my context had only `overview.md:676`.
3. Grep for `prompt_cache_ttl` lines mentioning validation (F7), because my context had only `template.yaml`.
4. Grep for `startsNoTurn`, `clearBoxBeforeCommand` and `ErrCommandTakenAsPrompt` repo-wide (F1 and F2), to find callers and stale comments.
5. `sed` of `claudeengine/doc.go` lines 94–102, plus a grep for `OpenLaunch` in that file, to judge the F6 twin.
