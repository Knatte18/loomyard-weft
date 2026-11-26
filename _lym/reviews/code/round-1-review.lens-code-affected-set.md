```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** I checked everything outside the touched lines that the diff's changes reach, and whether the diff leaves any of it wrong. I found two LOW findings.

**What I checked, per change**

1. **`readClientFormatsVia` (new) and the moved reads in `switchClientVia`, `buttonStrand` and `clientAgentPaneVia`.**
   - External callers: `internal/reedcli/switch.go:66` (`SwitchClient`), `internal/reedcli/button.go:79` (`RunButton`) and `internal/orchcli/remotecontrol.go:195` (`ClientAgentPane`). All are covered; none parses the error text.
   - Leftover `display-message -p -c` reads in `internal/reedengine`: none.
     The remaining `display-message -c` calls (`buttonclick.go:96`, `slacktoggle.go:24`) only show messages.
     The remaining `-p` reads without `-t` read server-wide formats, at `stopserver.go:22`, `tmuxconf.go:69/146` and `bindings.go:243/278`.
   - Docs: `reedengine/doc.go` is updated, and the `SwitchClient` and `buttonStrand` doc comments are updated. Covered.
   - Tests: `remotecontrol_test.go:40-44` asserts substrings that still match. Covered.
   - The error text `ClientAgentPane` returns changed shape, and the summary says it did not (finding 1).
2. **`SwitchClient` error contract** ("a client the server does not know" became "a client that is not attached"). The `reedcli/switch.go` help text does not describe it. Covered.
3. **New `session-bus` row (`SwitchNone`, `Forms`).**
   - Claude settings golden and the settings tests' counts are updated. The codex engine derives its rows from the table.
   - The `Forms` feed `cmd/lym/norecoverydeny_test.go:42`. No scanned stencil, spec or skill names those forms.
   - `shellguard/doc.go` is updated. `claudeengine/doc.go:252-260` defers to shellguard for each row and needs no edit.
   - The `machine-sleep` row's comment is corrected for busctl and dbus-send.
   - Covered.
4. **New `hub-tmux-server` row.**
   - The same consumers as the `session-bus` row are covered.
   - The sampler stencil, the one launch spec that sets `AllowRawTmux` and so the only session this row newly reaches, is left as it was (finding 2).
   - The sandbox suites (`tools/sandbox/SANDBOX-*.md`) run `tmux -L &lt;socket&gt;`, but lym-launched sessions already refused raw tmux. No new breakage.
5. **`shellguard` test imports `reedengine`** (`rows_integration_test.go:15`).
   - `PATTERN-leaf-packages` lists shellguard's closed import set.
   - The repository's own enforcement, `TestLeafInvariant_AllowlistOnly` (`internal/shellguard/rows_test.go:20`), scopes it to non-test files, and the package's tests already import testkit packages.
   - The production imports and the doc's import sentence (`shellguard/doc.go:74`) still hold. Not a violation.
6. **Commit messages.** The reed commit carries a `Root-Cause:` trailer that names the defect the diff removes. There are no `Pattern-Departure:` or `Loosened-Check:` trailers, and none is needed: the settings-test counts were raised to follow the new rows, not loosened.
7. **Declared PATTERN changes.** None are declared. Nothing to check.

**Findings**

1. **LOW · consistency · `_lym/landing/summary.md` ("What changes", RC bullet).**
   The summary claims `clientAgentPaneVia`'s "argv and errors are unchanged". They are not.
   - The not-attached and unreadable-row errors are now wrapped in the `reed: read the active pane of client %q:` prefix. At the base they were the bare `reed: no client %q is attached` and `reed: client %q has an unreadable list-clients row`.
   - The client name now shows twice.
   - This text reaches the operator's status bar through `ShowToggleOutcome`.
   - Evidence: the diff at `internal/reedengine/remotecontrol.go:38-41` against the removed lines 247-257; `internal/orchcli/remotecontrol.go:195-204`.
2. **LOW · consistency · `contracts/stencils/sampler/sampler-template-agent.md:43`.**
   The sampler is the one session the `hub-tmux-server` row newly reaches.
   - Its stencil tells the agent to start `tmux -L &lt;name&gt;` "with a name that holds the slug", and does not forbid a name opening with `lym-`.
   - A name such as `lym-&lt;slug&gt;-trial` is now refused, and the steer then points at lym's verbs, which is a wrong way forward for a trial server.
   - The row's comment says those servers "carry other names" (`internal/shellguard/rows.go`, new row comment), but the stencil does not guarantee it.
   - Fix: the stencil names the prefix the trial server must not open with.

**Tool calls**

1. Grepped claudeengine and codexengine docs, shellguard `doc.go` and `docs/reference` for lists of installed denies, and reedengine for leftover `display-message -p -c` reads. My context held neither.
2. Read `claudeengine/doc.go:240-275` and `shellguard/doc.go:30-66`, and grepped reedengine's `display-message -c` calls and the docs that name the bus commands. I needed the exact text of those deny descriptions.
3. Grepped reedengine's `display-message` reads, the external callers of the changed verbs, and the sampler stencil's naming instruction. The call sites outside reedengine and the stencil text were not in context.
4. Read `remotecontrol_test.go`'s error rows, the `orchcli` consumer of `ClientAgentPane`, and `reedcli/switch.go`'s help text. I needed to check the error-text claim against its consumers.
