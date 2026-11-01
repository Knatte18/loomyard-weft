```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, fallback and next step the fix introduced can be reached and leads somewhere useful.

Ways forward the fix introduced:
1. D8 rationale: CANCEL on a task whose beam is being landed is refused by `ResolveBattenPrime`, and the refusal reaches the operator as D7's failure message.
2. D4: a task whose parent beam has no anchor directory on this machine leaves out its `{beam}` entries, with one Warn.
3. D7: the run directory is the prime's anchor when the task's beam is absent.
4. D7: `lym reed button` exits 0 on every path once it has reported through `display-message`.
5. D8: `agent` runs `cd {worktree} && …`, so it opens in the task's worktree although the run directory is the beam's anchor.

Items 1, 2, 3 and 5 can be reached and make progress:
- **Item 1:** the refusal text names the worktree to re-run from. The operator can act on it from a terminal.
- **Item 2:** after the fix no default entry uses `{beam}`, so the Warn only ever touches entries a config author added.
- **Item 3:** `ResolveBattenPrime` accepts the prime, so CANCEL from the prime's anchor is not refused for its location.
- **Item 5:** `&&` and single-quoted `'opus[high]'` work under both bash and pwsh 7.

Findings:

- **LOW** — D7, "`lym reed button` exits 0 on every path once it has reported through `display-message`". This leaves open the path the same section names right after: a failed spawn or non-zero exit whose `display-message` itself fails because "a client or worktree gone by then", logged at Debug. On that path the verb has not "reported through `display-message`", so its exit code is unstated. A non-zero exit there opens view mode, which is the outcome F3 meant to rule out. The session usually goes with CANCEL, but a gone client in a surviving session does not. Evidence: D7's exit-0 bullet against its "only fails that message, logged at Debug" bullet.

- **LOW** — D7, the run directory "is the beam's anchor (`{beam}`) in a task session". This value is rendered at boot and attach (D5), but the beam is a different worktree with its own lifetime. If the beam is landed and removed while the task session lives, the click's `run-shell -c` names a directory that is gone, and nothing re-renders `@lym_button_dir` until the next attach. Every command then runs from tmux's fallback directory, outside any anchor, and its lym verbs are refused with `ErrCwdOutsideAnchor`. That refusal's message points at the cwd, not at the fix (re-attach the session). The fix's own presence check (D4) covers only absence at render. Before the fix, `#{session_path}` was the task's own anchor, which lives as long as the session. Evidence: D4's presence bullet ("has no anchor directory on this machine") applies at render only; D5's "A config edit … reaches a live session at its next attach or boot".

Out of scope, noted for the fix-claims lens: "The verb's log therefore lands in that anchor's logs" may not hold. The follow branch's `run-shell` uses no `-c`, and `cmd/lym` does no Getwd, so where lym's log goes may not depend on the verb's cwd.

Tool calls:
- grep of `internal/reedengine` bindings and follow files for `session_path` and `-c`: to see whether the follow model's `run-shell` sets a directory, which my context did not show. It returned nothing.
- grep for `run-shell` in `internal/reedengine` and for Getwd in `cmd/` and `internal/reedcli`: to see whether the verb process needs an anchor cwd. My context did not show how lym's entry point resolves cwd.
