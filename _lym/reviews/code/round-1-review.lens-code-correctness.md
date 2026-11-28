```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I traced each function and branch the diff adds or changes through one concrete case, including the failure paths. I found no wrong result, swallowed error or inconsistent state; there are two LOW findings.

**Functions and branches, the case run, the verdict**
- `holdInjectBuffers` (tmuxrec.go):
  - nil args give `[]`, true: correct.
  - The `load-buffer … ; paste-buffer -b <same name>` call gives two placeholders, true: correct.
  - Two different names give nil, false: correct.
  - `lym-inject-` with no hex is left as it is and matched literally: correct.
- `Replay.tmux`, the two-buffer branch: it takes the lock, `refuseLocked` records the raw args, and it returns "" with an error. `r.next` is read under the lock. Correct.
- `Replay.tmux`, the one-buffer branch: the held args go to `answer`, where `matches` compares them with the recorded `{inject-buffer}`. `bind` leaves that placeholder alone, since it is not a binding key and does not match the pid pattern. Correct.
- `Call.matches` with Stdin: the replay's reads carry no stdin. With `Stdin` absent the old recordings decode to "", so their calls still match a nil-stdin call. Correct.
- `bindCall` / `placeholdCall` on Stdin:
  - Bound paths round-trip: correct.
  - Numbers are kept on the record side. On the replay side `bind` also expands `{pid:N}`, which free text never holds. Correct in practice.
  - Bound values in free text are rewritten: finding 1.
- `placeholdCall` with `lym-inject-4711abcd` and pid 4711: the buffer is held before the pid step, so the result is `{inject-buffer}` and not `lym-inject-{pid:1}abcd`. Correct.
- `refuseLocked`, called from `answer` and from `tmux`: both hold `r.mu`. The message is unchanged. Correct.
- `describe` with Stdin: the `< %q` form. Correct.
- `Recorder.tmux` with nil stdin: `Stdin` is "" and omitted from the JSON. Correct.
- `newSession` when `ResolveShellPath` fails: it binds the bare `cfg.Shell` and the session builds. For PlaySession the OS skip comes first. Correct, with finding 2.
- `IdleSession` with an unknown name: `Fatalf`. With a known name it returns the session and drops the replay. Correct as documented.
- `replayedSessions` (envkit) when register is called: it writes under `dir/runproc`. It is not driven by any caller. Correct.
- `reedengine.InjectBufferPrefix`: a pure extraction, with the same output. Correct.
- `prReviewTestDeps` MailAgent (landingshed): if it is called, it reports `t.Errorf` and returns an error. The row then halts and the test fails. Correct.
- `TestPRReviewGateFailureMail`:
  - Row 1: gated on the first wake, mailed, and the next two wakes skip because `MailedHead == head`. One gate, one mail. Correct.
  - Row 2: cleared at sleep 1, gated and mailed again on the next wake, skipped on the third. Two gates, two mails. Correct.
  - Row 3: creel refuses `pr-gone`, `mailFailure` returns false and `halt` gives the reason with `logs[0]`. MailedHead stays "". Correct.
  - Indexing `logs[0]` is safe because every halt row has `wantGates ≥ 1`.
  - The inner `err :=` is legal because it introduces new names.
  - Only the entry poll runs: 1+2+4 s of sleep stays under the 60 s poll interval, so one scripted status is enough. Correct.
- `prReviewMailDeps`: the `gates` counter is captured by pointer. The seeded `StartSession` takes the recorded start of `session-start-stop`. Correct.
- `TestReplay_MatchesAPasteByItsTextWhateverBufferItNames`: each row is refused for exactly the reason in its name. The one-name row matches, the text row is refused by Stdin, and the two-name row is refused by `holdInjectBuffers`. Correct.

**Findings**
1. LOW, design. `internal/testkit/tmuxrec/record.go:590`, where `placeholdCall` sets `held.Stdin = bindings.placehold(call.Stdin)`. Every bound value is replaced in a paste's free text, including `{user}`, `{host}`, `{tmp}` (`os.TempDir()`, such as `/tmp`) and `{socket}`. A pasted text that happens to contain one of those strings, such as a literal `/tmp/...` or the operator's user name, is stored as a placeholder. A replay on another host binds it to that host's value while the code still sends the literal, so the paste goes unanswered. `checkNameBindings` (record.go) guards only the session's fixed names and never the stdin text. Evidence: record.go:590, `bind` at tmuxrec.go:221–228, the bindings at session.go:96–110.
2. LOW, design. `internal/testkit/tmuxrec/session.go:120`, the bare-name shell fallback. `newSession` also serves `RecordSession`. On a host without the shell, `{shell}` is bound to a short bare name such as `bash`. `Recorder.Recording` then runs `bindings.placehold` over every Out and Arg, so every `bash` substring becomes `{shell}`, including `pane_current_command bash` and the `-V`/`list-commands` output. A recording written there is silently corrupt instead of refused, and no `checkNameBindings`-style guard covers `{shell}`. Evidence: session.go:117–121, record.go `Recording`/`placeholdCall`, tmuxrec.go:203–215 `placehold`.

Tool calls:
- Grep for reedengine tmux calls with non-nil stdin, to confirm that only the paste hands tmux stdin. My context had no reed call sites.
- A second, wider grep of reedengine `Tmux(` call sites, for the same reason. It was not conclusive; the tier0 replay over every recording covers the question.
