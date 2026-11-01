```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, key, string, file shape or behaviour a card changes, I list what reads or asserts it and which cards cover that set.

**Per change: the set found and the cards that cover it**
- The `Shell` interface gains `DiscardOutput`, `ExitSuccess` and `CommandFlag`. Its only implementers are `posixShell` and `pwshShell` (`internal/shell/posix.go`, `internal/shell/pwsh.go`). Card 1 covers the set.
- `desktophook.Config` gains a `yaml.Node` field, so the struct is no longer comparable. The only comparison is `internal/desktophook/config_test.go:63` (`got != tt.want`). Card 2 covers it.
- `reedengine.Config` gains `Buttons`. Its literals in `internal/loomcli/start_watchdog_test.go:32` and `internal/webstercli/cli_tmux_test.go:78` set no `Buttons` and need no change. Card 2 covers the set.
- `bindingArgvs` is re-signed. Its callers:
  - `pinBindingsLocked`;
  - `TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` (`bindings_test.go:34`, `:55`, `:70`, `:87`, `:106`, `:120`, `:126`);
  - `bindKeyArgvCount` (`windowsize_test.go:29`);
  - the `TestPinGeometryOptionsLocked` call (`windowsize_test.go:154`).

  Card 9 covers all of them.
- `switchBindingCommandsLocked` is re-signed. Its only caller is `pinBindingsLocked` (`bindings.go:140`). Card 9 covers it.
- The session's `status` goes from 2 to 3. Readers and asserters:
  - `windowsize.go:76`;
  - `windowsize_test.go:47`, `:51`, `:65`;
  - `bindings_integration_test.go:343`;
  - `internal/reedcli/smoke_selvage_keepalive_test.go:273` (comment) and `:338` (`got != "2"`).

  Card 5 covers every member but the smoke test, which is **missed**.
- `status-format[1]` loses the follow button. Readers and asserters:
  - `bar.go`;
  - `bar_test.go:46`;
  - `windowsize_test.go:63`;
  - `bar_integration_test.go`;
  - `bindings_integration_test.go`;
  - `tmuxconf_integration_test.go:264`, which asserts only a session button, so it stays true.

  Card 5 covers the set.
- The follow button's place on the bar is described in `reedcli/follow.go:2` and `:40` and `reedcli/doc.go:28`. These say "the status bar's follow button" with no line number, so they stay true and need no card.
- The claim that `@lym_follow` is the one decision-bearing option changes. Statements of it:
  - `bar.go:9`;
  - `follow.go:2`;
  - `doc.go:79` and `:381`.

  Card 6 covers these.
- The new `@lym_strand_name` makes a strand pane option decision-bearing. Statements that no Go decision reads the strand options:
  - `paneoptions.go:1-3` (file header);
  - `doc.go:78`.

  Card 7's Intent names both, but its targets list only the member `Engine.setStrandPaneOptionsLocked` (`paneoptions.go:28`) and `doc.go`. The header at lines 1–3 lies outside that member's span, so it is **uncovered by targets**.
- Card 8 adds `composeConfirmedButtonLine` to `panebin.go`. The two tests in `panebin_enforcement_test.go` (`:64`, `:103`) scan only for pane-creation sites (`split-window` literals), so the new line does not trip them. No card is needed.
- The new verb `reed button` touches:
  - the `PersistentPreRunE` skip list (`reedcli/cli.go:142`);
  - the `AddCommand` list (`reedcli/cli.go:189`);
  - `cmd/lym/stencilseedgate_test.go`;
  - `reedcli/doc.go`.

  Card 8 covers the set.

**Findings**

1. **MEDIUM**, class `design`.
   - Location: card 5 (`third-status-line`).
   - Summary: card 5 changes `status` from 2 to 3 but misses `internal/reedcli/smoke_selvage_keepalive_test.go`. Its subtest `StatusLinePinsIdentityAndPosition` asserts `readOption("status") != "2"` at line 338, and its comment at line 273 says `#{status}` reads "2" (the two-line bar). After card 5, that tier2 test fails.
   - Fix: list that test's member glyph on card 5's `**Edit:**`.
   - Evidence: `internal/reedcli/smoke_selvage_keepalive_test.go:273`, `:338`; `internal/reedengine/windowsize.go:76`.
2. **LOW**, class `consistency`.
   - Location: card 7 (`strand-name-pane-option`).
   - Summary: card 7's Intent rewrites `paneoptions.go`'s file header, which says "No Go decision reads them back" (lines 1–3). Its targets list only `internal/reedengine#Engine.setStrandPaneOptionsLocked`, whose span starts at line 28 and does not hold the header.
   - Fix: name the file's self glyph `internal/reedengine/paneoptions.go` in place of the member glyph. Listing both would trip `redundant-file-target`.
   - Evidence: `internal/reedengine/paneoptions.go:1-3`, `:28`.

Tool calls:
1. Grepped for `Shell` implementers, `Config` literals and comparisons, `bindingArgvs` and `switchBindingCommandsLocked` callers, and `status 2` / `status-format[1]` readers. The manifest held only the definitions, not their repo-wide readers.
2. Mapped `bindingArgvs` test call sites to their test functions, read `desktophook/config_test.go`'s comparison, and listed files that mention line 2 or the follow button. I needed the enclosing test names and the doc readers outside the pre-read.
3. Read the line-2 and two-line-bar mentions in reedcli and the integration tests, `desktophook` Config comparisons, and display-only / decision-bearing doc statements. These are doc and test readers of the changed strings, which the manifest lacked.
4. Read the smoke test's status comment, `paneoptions.go`'s header and member positions, and listings of the `reed follow` verb. I needed the exact header span and the verb's listing sites.
5. Read the smoke test's Windows branch and `panebin_enforcement_test.go`'s scope. I needed to know whether either asserts `status` or composed lines.
6. Grepped for the smoke test's exact `"2"` assertion line and any status-row count in production code, to confirm the missed assertion.
