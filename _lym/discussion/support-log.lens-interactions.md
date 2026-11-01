```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, then checked each acceptance criterion and constraint against the decisions it touches.

**Findings**

1. **MEDIUM, D2 × D5 (validation and the attach render).** D2 puts button validation where `validateWindowConfig` runs, so an invalid entry fails up, resume and add. D5 renders the row both at boot and in the attach pre-flight, and the attach path runs no validation.
   - Evidence: `validateWindowConfig` is called in `lifecycle.go:313`, `strand.go:574`, `replace.go:52` and `ownwindow.go:287`. `attach.go:98` calls `pinGeometryOptionsLocked` and never validates.
   - Effect: a `reed.yaml` edited to an invalid shape after boot is rendered at the next attach. For example, `{strand}` without `window: agents` lands in the spec unfilled, and a label holding `#` reaches `#{E:}`.
   - The record says nothing about this path. D5's render should apply D2's rules itself, for example by skipping the entry with a Warn as D3 does.

2. **MEDIUM, D7 (internal contradiction about the shell).** D7 says the verb "reads no working directory, config or reed state, only the session's spec option". It also says the verb runs `run` "through reed's configured shell". That shell is reed's `shell` config key, and D6 quotes values for that same dialect at render time. Neither D5's spec contents nor the pinned click line carry the shell. Fix: add the resolved shell path to the spec (D5) or to the pinned line.

3. **MEDIUM, D7 confirm path × D6 quoting.** D7 passes "the entry's filled `run`" inside the `confirm-before` command, which re-runs the click line under `run-shell`. That text then goes through tmux command parsing and `run-shell`'s format expansion.
   - D6's values are quoted for the shell, and a `run` may hold `#` (bash `$#`, comments) and nested quotes. Each of these is mangled or expanded by tmux before the "spec unchanged" comparison.
   - So a legitimate confirmed click can fail D7's own spec-changed check, or run something other than what was drawn.
   - Fix: carry a digest of the entry (index plus a hash of `run`) across `confirm-before`, never the `run` text.

4. **LOW, D7 (log location versus what the verb resolves).** The trace file is meant to land in the worktree's `.lym/logs/` because the verb "runs in the worktree". The logger resolves `LogsDir` from the process cwd when it writes its first record. `run-shell -b` starts in the server's cwd, and the verb learns the worktree only from the spec option. Any log record before a chdir opens the sink elsewhere. Fix: state that the verb chdirs to the spec's worktree before its first log record, or have the pinned line set the start directory.

5. **LOW, D2 × D6 (`#` in filled values).** D2 bars `#` in the `confirm` template, and D6 then fills values into `confirm` plain. `{worktree}`, `{beam}` and `{session}` can hold `#`: hub paths are unrestricted, and `validateToldTmuxIdentity` bars `.`, `:`, `\` and control characters, not `#`. `confirm-before -p` expands status-left sequences, so a filled `#` reopens what D2 closed. Fix: check after filling, or escape filled values in `confirm`.

6. **LOW, D1 (status pins from an older binary).** `bindingsStale` stops an older binary from re-pinning the bindings, but the status pins have no such guard. An older lym attaching anywhere on the server re-pins the global `status-format[1]` with `follow` in it. Every new session then shows `follow` on both line 2 and line 3 until a new binary's next attach. The record should accept this transitional state in words, or bring the status-format pins under the build stamp.

**Pairs checked**
- D1 × D5 (`status-format[2]` and the row options, both set at the status pins): held.
- D1 × D7 (follow branch and `lymbtn*` branch in one dispatch): held, the matches are disjoint.
- D1 × D4 (standalone shows `follow` alone): held.
- D2 × D3 (prime fallback per source, order reed then machine): held.
- D2 × D3 (validation fails for reed entries, skips machine entries): held as designed.
- D2 × D5 (label `#` bar against `#{E:}` and the conditional): held when validated; broken on the attach path (finding 1).
- D2 × D6 (`#` bar in `confirm` against filled values): broken (finding 5).
- D2 category × D3 local refusal × AC D3: held.
- D3 × the `PATTERN-config-layers` constraint (machine layer needs Anchor and Board): held, as long as the closure passes both.
- D4 × D5 (closure failure gives `follow` alone, spec still rendered): held.
- D4 × D8 (`{slug}` and `{beam}` values against the default commands in the prime, a beam and a task): held.
- D5 × D7 (spec drawn equals spec clicked; refusal when it changed): held in design; broken in transport (finding 3).
- D5 × D7 (the spec carries everything the verb needs): broken for the shell (finding 2).
- D6 × D7 (`{strand}` filled at click, quoted): held.
- D7 env removal × D8 (HALT reads the click as external, `shuttle open` and orch verbs): held.
- D7 cwd × D8 CANCEL (`cd {beam}`): held.
- D7 log × cwd: broken (finding 4).
- D1 pins × older-binary pins: degraded (finding 6).
- AC D1, D4, D5, D6 and D8 against their decisions: consistent.
- AC D7 against D7: holds only once findings 2 and 3 are fixed.
- Constraints (tmux 3.4 constructs, psmux, shell guard, spawn logging): consistent with the decisions.

**Tool calls**
- One grep for the `validateWindowConfig` and `pinGeometryOptionsLocked` call sites. My context held the attach pin but not whether attach validates.
