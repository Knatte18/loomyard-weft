```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, this is what depends on it and whether the diff covers it.

**Per change, the set found and whether it is covered**

1. **Clear before paste (`pasteVerified`, `clearBoxBeforeCommand`, `InputBoxClearer` contract)**
   - The paste path of `sendSequence` is reached by clear, compact, reload-plugins and color, from `creelcli/delivery.go:137-143`, `session.go:386` and `run.go:770` (`typeStartupColor`).
   - The doc updates are covered:
     - `InputBoxClearer` in `engine.go`
     - `shuttleengine/doc.go`
     - `claudeengine/doc.go`
     - the Sequence comments in `claudeengine/session.go`
   - Tests asserting that no keys are sent were not rechecked: `enginedispatch_test.go:668` and `:712`, and `run_inject_test.go:52`, `:98`. They look like fixtures for non-paste engines or a refused-open path.
   - Covered except for those unchecked tests.

2. **`ErrCommandTakenAsPrompt` and `mapDeliveryError`**
   - `creelengine/deliver.go:425-430`, the default branch of `typeInject`, requeues the inject and reports the error through `report("send", err)`. That matches the change description.
   - The orch watcher's color path (`orchengine/watcher.go:796-846`) reads the inject outcome. A requeued color just stays pending, so it is covered.
   - No missing handler found.
   - The creel docs are not updated. See finding 1.

3. **`OpenLaunch` TTL export (`ExportEnv` + `Chain`)**
   - The only consumer of the `Launch` lines is `shuttleengine/open.go:111-112`. It wraps them again with `sh.Chain(...)`, which is shell-compatible, since both dialects join with `; `.
   - The `codexengine` `OpenLaunch` is unaffected because it sets no TTL.
   - `claudeengine/doc.go` is covered.
   - These two places are not updated. See findings 2 and 3:
     - `docs/overview.md:676`
     - `template.yaml:3-4`

4. **PATTERN and declared changes**
   - The diff touches no `PATTERN.md` or `pattern/` file, and I saw no declared changes list in the constraints section I read, so nothing is owed there.
   - The shuttle provider-seam entry is respected, because the new code sits under `claudeengine` and in `shuttleengine`.
   - `PATTERN-code-conventions`: the log message and key reuse `promptCacheTTLEnvKey`, so there is no new literal.
   - Commit trailers: none of the four commits carries a `Pattern-Departure:` trailer, and the summary says `pattern_departures: none`. Nothing to check.

**Findings**

1. **LOW, consistency** — `internal/creelcli/doc.go:149-150` and `internal/creelengine/doc.go:409-412` describe the command inject's error contract as only ErrBusy and ErrNotLanded.
   - A command taken as a prompt is now a third outcome. It is requeued and retyped, and it is reported as a send failure.
   - Only the `mapDeliveryError` comment records it.
   - Evidence: `creelcli/doc.go:149-150`, `creelengine/doc.go:412`, `creelcli/delivery.go:460-461`.

2. **LOW, consistency** — `docs/overview.md:676` says the engine sets the TTL "on launch and resume".
   - Opened sessions now get it too, through the open path.
   - The sentence is now incomplete for the repository's overview doc.
   - Evidence: `docs/overview.md:676`, `claudeengine/command.go:489-494`.

3. **NIT, consistency** — `claudeengine/template.yaml:3-4` says the TTL is "validated when a run starts".
   - `OpenLaunch` now also validates it, and an invalid entry refuses the open.
   - Evidence: `template.yaml:3`, `command.go:454`.

**Coverage gap, not a finding in this lens:** `internal/creelcli/delivery_test.go:407-409` and `:461` have no row mapping `ErrCommandTakenAsPrompt` through `mapDeliveryError`. The `code-tests` lens owns this.

**Tool calls**
1. `grep` for `prompt_cache_ttl` / `PROMPT_CACHE_TTL` in non-test files, to find docs and templates that describe the TTL scope. My context did not hold them.
2. `grep` for callers of `ClearSession`, `CompactSession`, `ReloadPlugins`, `TypeColor` and `typeColor`, to find the set that reaches the paste path.
3. `sed` of `creelcli/doc.go` and `creelengine/doc.go`, plus a `grep` of `.md` files for prose about clearing. This checked the docs that describe the error contract.
4. `sed` of `orchengine/watcher.go`, `run.go:760-785` and the `creelcli/delivery.go` dispatch, to see how color and command errors are consumed.
5. `grep` of the test files for "no key" assertions and for the `ErrCommandTakenAsPrompt` mapping row. I did not read the tests it listed, so the assertions at `enginedispatch_test.go:668` and `:712` and `run_inject_test.go:52` and `:98` are unchecked.
