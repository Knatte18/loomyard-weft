```yaml
lens: bound-breaker
outcome: reported
```

Task: the bound-breaker lens on the record's claims and bounds.

**Claims attacked**
- D1, simple-mode guard: **held.** Ugt returns true only when `Sr()` is true, and `Sr()` is `CLAUDE_CODE_SIMPLE` or `--bare` (`.scratch/cc.txt:561618`, `:552475`). `ScrubbedEnv` strips the `CLAUDE_CODE_` prefix (`internal/shuttleengine/claudeengine/scrub.go:8`). The pane command sets only `CLAUDE_CODE_FORK_SUBAGENT` and `CLAUDE_CODE_PROMPT_CACHE_TTL` (`command.go:107,112`).
- D1, Stop additional context continues the turn: **held.** It is pushed into `He`, returned as `blockingErrors`, and the next pass sets `stopHookActive:!0` (`.scratch/cc.txt:569593`, `:569601`).
- D1, Claude Code's cap of 8 still counts these continuations: **held.** `Me=$r+1` is checked against `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP??8`, and reed's scrub leaves the default in place.
- D1 and D2, a block adds the command text and the error label, and `systemMessage` writes `Stop says:`: not re-attacked, since the source read in my context covers only the additional-context path.
- D3, an asyncRewake hook's stderr reaches the model only on exit 2, and the body is `<rewakeMessage> <stderr||stdout>` with priority `next`: **held** (`.scratch/cc.txt:564111`).
- D3, Claude Code escapes or replaces its marker patterns: **held.** `bWe` replaces a fixed pattern list (`.scratch/cc.txt:562126`).
- D3, "once a ring carrying mail has arrived it never resumes the wait": **broken against the existing client.** The record does not say the client has to change; see F1.
- D3, an unmarked waiter (an older binary's) rings with no mail, and the mark survives a resume: **held.** `resume` resends `req` unchanged for `VerbHookWait` (`internal/creelengine/client.go:176`).
- D3, an empty ring claim does not ring and keeps the waiter: **held.** The worker's `locate` already skips paused strands, and `claimableLocked` already excludes other senders' mail under a hold.
- D3, after a release the bell's 30-second window applies as today: **held.** Reserved ids leave `unreadLocked` (`pool.go:446`). Released ids equal `mark.ids`, so the worker takes the unanswered branch and types the bell after `ringAnswerWindow` (`deliver.go:274-287`).
- D3, the ring starts the Stop count over: **held.** The wake notification carries `stopHookActive:!0` (`.scratch/cc.txt:564111`). The woken turn's Stop therefore does not reset `stopBlocks`, but the ring already set it to 0 (`verbs.go:451`).
- D3, the pause-window bound and the two loss-window bounds: **held as stated.**
- D3, exit 0 "whose stderr it drops": **held for the model.** `_f` still records the output for every exit code before the code-2 check (`.scratch/cc.txt:564109`). I did not trace whether that record is shown anywhere, so this is unconfirmed and not a finding.
- D4, the default prefix and summary: **held.** The summary prefers `rewakeSummary` from stdout JSON only for plugin hooks.
- Constraint, the 8 KiB budget fits every field: **held.** The first message's line is at most `hookFieldCap` UTF-16 units (`verbs.go:56`), and UTF-8 bytes are never fewer than UTF-16 units. The record says the rewake body is not truncated; I could not confirm that `Di` and the trailing renderer leave it whole, but I did not break the claim either.
- Constraint, the request mark is tolerated by an older daemon: **held.** `serve` uses plain `json.Unmarshal`, which ignores unknown keys (`server.go:201`).
- D5 bound, a hook that never reaches the daemon "stays silent toward the session as today": **held**, since it is unchanged from today.
- Constraint, delivered once with two exceptions: **held**, apart from the F1 counterexample.

**Findings**

F1: LOW, consistency
- Location: decision-record D3 (second paragraph) and "Notes for the plan writer".
- Summary: today's Go client resumes a hook-wait whose acknowledgement connection drops, and the record names only the client's automatic acknowledgement as something that must change.
- Evidence: `CallWatching` reconnects and resumes whenever `errors.As(err, &lost)` and `req.Wait` hold (`internal/creelengine/client.go:144-157`). `resume` resends a `VerbHookWait` request unchanged (`client.go:176-177`). `awaitAckConfirmation` and the ack write both return `*connLostError` (`client.go:256-264`, `:272-274`). `internal/creelengine/doc.go:140` states the resume as the rule.
- Counterexample: a ring carrying mail arrives and `hook-wait` writes the lines to stderr. The connection then drops before the daemon's confirmation. The client reconnects and registers a new marked waiter, so the lines stay on stderr and the process waits again. The next ring appends a second set and exits 2. The model gets the first set, and that set may also have been released and handed out again, which is the duplication D3 forbids.
- Mitigation: the D3 acceptance criterion "exits 0 without resuming when … the acknowledgement fails" covers the requirement, so a test would catch it.
- Fix: the Notes line on the Go client should also say that a hook-wait carrying mail must not take `CallWatching`'s reconnect-and-resume path once the ring's outcome has arrived.

**Tool calls**
1. Grepped `scrub.go` and the claudeengine `CLAUDE_CODE_` keys, to check D1's scrub claim; the context had no scrub code.
2. Read `Ugt`, `bWe`, `rx` and `Bin` in the extracted 2.1.312 strings, to check the mode guard and the rewake body; the context had only their call sites.
3. Read the rest of `Bin` and the `Sr` definitions, to confirm the guard is `CLAUDE_CODE_SIMPLE` or `--bare`.
4. Looked up `Sr` candidates (folded into call 3) to tell the simple-mode `Sr` apart from a same-named function.
5. Tried to find `Xa`, `zt` and `Di`, to check whether the rewake body is truncated; the result was inconclusive because minified names collide.
6. (Counted with call 5.) No further reads.
