```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each added or changed branch on a concrete input. I found one defect, in `composeButtonCommand` and `composeConfirmedButtonLine`.

**Per-branch list** (Chain joins with `; `, so every statement after a failed one still runs)

- `composeButtonCommand`, pinned binary missing: the redirect `&gt;/dev/null 2&gt;&amp;1` covers the shell's "not found" message, `exit 0` runs, status 0. Correct.
- `composeButtonCommand`, run directory gone (`cd` fails): INCORRECT, see the finding below.
- `composeButtonCommand`, `#` doubling and `#{q:}` words: pinned tokens are doubled and the dynamic words are left for the click's expansion. Correct.
- `composeConfirmedButtonLine`: same shape as the click line, with `ChangeDir` outside the redirect. Incorrect for the same reason.
- `tmuxQuote` into `run-shell -b "&lt;line&gt;"`: `\`, `"`, `$` and `#` are escaped, and `;` stays inside the quotes. Correct.
- `bindingArgvs` button branch and `pinBindingsLocked`: the line is set when it resolves and unset otherwise, the same as the follow line. Correct.
- `repinStaleBindingsLocked`: the global status formats are re-pinned, then the bindings. Per-session `status 3` is not re-pinned, which matches D1's stated bound. Correct.
- `runButtonVia`:
  - Unrendered spec (empty option): one message, nothing run. Correct.
  - Unknown range: message, nothing run. Correct.
  - Gone directory: checked before the prompt, message, nothing run. Correct.
  - Confirmed with a stale hash: refused with a message. Correct.
  - Confirmed with empty `req.Strand` on a `{strand}` entry: `agentname.Parse` fails, message. Correct.
  - Exit code -1 from a signal: reported as a failure. Correct.
  - Failing `display-message`: logged at Debug, no panic. Correct.
- `buttonStrand`, unconfirmed with an empty `@lym_strand_name`: message, nothing run. Correct.
- `readButtonSpec`: read failure, empty value and bad JSON each log a Warn and report false. Correct.
- `findButtonEntry`: `lymbtnX` and an index with no entry both report false. Correct.
- `placeButtons`:
  - Zero node or absent group: no entries, no Warn. Correct.
  - Mapping group: one Warn, skipped. Correct.
  - Wrong-typed entry: skipped with a Warn, the others render. Correct.
  - Prime with an empty list: falls back to `beam`. Correct.
  - Prime group that is not a list: Warns, then falls back to `beam`. Acceptable under D2.
- `buttonEntryProblem`: empty label or run, `#`, `%`, control characters, bad `window`, and `{strand}` without `window: agents` are all refused. Correct.
- `fillRunPlaceholders` and `fillConfirmPlaceholders`: single pass, unknown brace text stays literal, the confirm value has `#` and `%` doubled. Correct.
- `composeButtonRowLocked`:
  - Nil `Buttons`: empty render. Correct.
  - `Buttons()` error: empty render. Correct.
  - Shell does not resolve: empty render. Correct.
  - Task place with an absent beam: run directory falls back to `PrimeAnchor`, and `{beam}` entries are dropped with a Warn. Correct.
  - Entries are renumbered after skips, so range N matches `spec.Entries[N]`. Correct.
- `pinButtonRowLocked`: the four options go in one invocation. An empty render sets all four to "". Correct.
- `buttonContext` and `buttonBeam`:
  - Origin read error: the whole context fails, so `follow` alone. Correct.
  - Desktop load error: only the machine buttons are dropped. Correct.
  - `ParentOf` returning "": the prime's anchor. Correct.
  - Parent anchor directory missing: `Anchor` empty. Correct.
- `setStrandPaneOptionsLocked`: sets `@lym_strand_name`, or unsets it for an empty name. `@strand` is set only here. Correct.
- `ConfigSpec` (`buttons` classified `CategoryExecutable`) and `ConfigOpenMaps`: not run, because the lens procedure did not need them.
- `RunButton` has no `recover`, so a panic exits non-zero. This is covered only by the forced `exit 0` in the line, as D7 intends.

**Finding**

1. **MEDIUM, class design:** `internal/reedengine/panebin.go:319` and `:339`. The directory change sits outside `DiscardOutput`. Both lines are `sh.Chain(ChangeDirToWord(...), DiscardOutput(verb), ExitSuccess())`, so only the verb is redirected.
   - Input: the run directory is gone, or `@lym_button_dir` is empty or unquoted in pwsh.
   - Step: `cd`, or pwsh `Set-Location`, writes an error to stderr.
   - Result: that text is output of the background `run-shell`, which D7 says opens view mode ("whatever the binary or verb does"). The forced zero status does not prevent this. D7 lists "a gone run directory" as a case that must not open view mode.
   - Impact: the verb's friendly "directory is gone" message (`buttonclick.go:96`) is shown only after the stray output has already opened view mode.
   - Fix: put the whole chain, `cd` included, inside the discard, for example `sh.DiscardOutput` over a grouped `cd &amp;&amp; verb`. Alternatively, redirect the `cd` too.

**Minor, not filed** (LOW; cheap to note): `validateButtonFlags` failure (`internal/reedcli/button.go:62`) calls `SetExit` with a non-zero code. D7 says the verb exits 0 on every path, but the forced `exit 0` in the pinned line covers the click path. It is reachable only by a direct call.

**Tool calls** (3 of 6)
1. `sed` on `posix.go` and `pwsh.go`, plus a grep for `@strand` set sites: my context lacked `Chain`'s join semantics and the other `@strand` sites.
2. Read `clihelp.SetExit` and `exactSessionWindowTarget`: needed for the exit-code and session-target behaviour.
3. Grep for `LoadConfig` callers: to check whether attach reloads `e.cfg`. The result is inconclusive for the "config edit reaches a live session at attach" claim; only `reedcli/cli.go:169` loads the config, per command. Left to the other lenses.
